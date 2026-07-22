# Phase 2 기능명세 — 인증 · 북마크 · 마감임박 알림

> Phase 1(조회·제보·검수) 완결 후 착수. 배치 근거 [ADR-0003](../adr/0003-auth-last.md).
> **이 문서는 스펙 스케치** — 상세 시스템 설계·ADR은 실제 착수 시점에 Phase 1 운영 피드백 반영해 확정(현 시점 상세 설계는 재작업 위험).

## 목표

- 인증 도입으로 개인화 기능(북마크)과 알림의 수신 주체 확보.
- `/api/admin/**` 를 인증으로 잠가 의도적 부채 해소([ADR-0003](../adr/0003-auth-last.md)).

## 범위 (3 기능, 의존 순서)

1. 인증 (선행 — 나머지가 User에 의존)
2. 북마크 (인증 의존)
3. 마감임박 이메일 알림 (인증 + 북마크 의존)

---

## 1. 인증 (소셜 로그인)

**결정**: **OAuth2 소셜 로그인(Kakao·Google)** 으로 구현. 자체 비밀번호 미보유.
- 기각 — 이메일·비밀번호 자체 인증: 비밀번호 저장·유출 책임·재설정 흐름 부담. 국내 대상엔 카카오 로그인이 가입 마찰 최저. 소셜만으로 시작.

**사용자 스토리**
- 방문자는 Kakao/Google 계정으로 로그인(별도 가입 폼 없음).
- 로그인 사용자는 제보 시 제보자 정보가 계정에 귀속.
- 운영자(ADMIN)만 검수 큐·검수 API 접근.

**엔티티**
- `User`: id, provider(KAKAO/GOOGLE), providerId(공급자 계정 고유 id), email(**nullable**), displayName, role(USER/ADMIN), createdAt.
- `(provider, providerId)` unique. **비밀번호 필드 없음.**
- 최초 로그인 시 공급자 프로필로 User 자동 프로비저닝.

**엔드포인트(안)**
- Spring Security OAuth2 Client 표준 흐름: `GET /oauth2/authorization/{kakao|google}` → 공급자 리다이렉트 → 콜백 `/login/oauth2/code/{provider}` → 세션 수립.
- `POST /api/auth/logout`, `GET /api/me`.

**규칙**
- Spring Security로 `/api/admin/**` = ADMIN, 제보/개인화 API = 인증 필요, 공개 피드/상세 = 무인증 유지.
- **ADMIN 승격은 자가가입 불가** → 운영자 계정을 허용목록(providerId) 또는 DB 시드로 수동 부여.
- 현재 유저는 `SecurityContext` 직접 접근 금지 → `CurrentUserProvider`류 주입(프로젝트 컨벤션·전역 규칙).

**설계 주의**
- **Kakao email 비보장**: 카카오 email 제공은 선택·심사 대상 → email을 식별자로 쓰지 말 것. 키는 `(provider, providerId)`, email은 nullable 부가정보(알림용). Google은 email scope로 보통 제공.
- **계정 연결(linking)**: 같은 사람이 Kakao·Google로 각각 로그인하면 별도 User 2개. 통합은 복잡도 커서 defer.
- 공급자 프로필 수신 → 개인정보 동의 화면은 공급자가 처리. 우리 수집은 최소(식별자·표시명·email만).

**마이그레이션 영향**
- `Event.submitterName`/`submitterEmail`(텍스트) → `submitted_by_user_id` FK로 승격. 기존 익명 제보 데이터는 nullable FK로 하위호환([ADR-0002](../adr/0002-moderation-on-event.md), [ADR-0003](../adr/0003-auth-last.md)).
- Flyway 신규 마이그레이션으로 `users` 테이블(`(provider, providerId)` 유니크) + `events.submitted_by_user_id` 추가.

---

## 2. 북마크

**사용자 스토리**
- 로그인 사용자는 관심 이벤트를 북마크/해제, 내 북마크 목록 조회.

**엔티티**
- `Bookmark`: user_id, event_id, createdAt. `(user_id, event_id)` unique.

**엔드포인트(안)**
- `POST /api/events/{id}/bookmark`, `DELETE /api/events/{id}/bookmark`, `GET /api/me/bookmarks`.

**규칙**
- 북마크 목록도 이벤트 상태는 저장 안 하고 파생([ADR-0004](../adr/0004-derived-status.md)) — 목록 조회 시 N+1 주의(배치 로딩).

---

## 3. 마감임박 이메일 알림

**사용자 스토리**
- 북마크한 이벤트가 마감임박(ENDING_SOON, endAt 48h 이내) 진입 시 이메일 1회 수신.

**메커니즘**
- `@Scheduled` 배치(예: 시간당)로 ENDING_SOON 진입 이벤트 조회 → 해당 이벤트 북마크한 User에게 발송.
- 상태는 파생이므로 배치는 시간 술어로 쿼리([ADR-0004](../adr/0004-derived-status.md)) — 별도 상태 컬럼 동기화 불필요.

**엔티티**
- `NotificationLog`: user_id, event_id, sentAt. `(user_id, event_id)` unique → **중복 발송 방지 게이트**(재실행·재시작에도 1회 보장).

**규칙**
- N+1 금지: 대상 이벤트·북마커·기발송 로그를 배치 조회 후 `groupBy`/`associateBy`로 조합.
- 발송 실패는 로그 미기록 → 다음 배치에서 재시도(멱등).

**소셜 로그인 교차 파급 (중요)**
- 카카오 로그인 유저는 email이 없을 수 있음(§1) → **email 없는 유저는 이메일 알림 대상에서 제외.**
- 완화(택1, 착수 시 결정): ① 알림 켤 때 email 별도 입력 요청, ② 공급자 email scope 적극 요청, ③ Kakao 알림톡 채널(별도 통합·비용 → defer).

---

## 열린 결정 (착수 시 ADR로 확정)

- **인증 전송 방식**: 세션 쿠키(HttpOnly, 서버 세션) vs JWT. OAuth2 리다이렉트 로그인은 세션 수립이 기본이고, 배포가 CloudFront 단일 도메인 경로 라우팅([ADR-0005](../adr/0005-deployment-target.md))이라 **동일 오리진** → 세션 쿠키(HttpOnly) 무난. 착수 시 확정.
- **알림용 email 확보**: 카카오 email 비보장(§1·§3) 대응 — 별도 입력 요청 vs email scope 적극 요청 vs 알림톡. 착수 시 결정.
- **이메일 발송 프로바이더**: Resend(개발 친화) vs AWS SES(저비용·확장, ADR-0005 AWS 스택과 정렬). 초기 Resend 권장, 물량 증가 시 SES 재검토.

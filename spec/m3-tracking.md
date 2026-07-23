# M3 기능명세 — 북마크 · 마감임박 알림

> 제품 정의는 [PRD](../prd.md). 이 문서는 **M3(추적)** 단계의 기능명세.
> 종전 `phase2.md`(인증·북마크·알림)에서 **인증은 구현 완료로 분리**하고, 남은 북마크·알림을 M3로 재배치한 문서.
> 착수 시 M1·M2 운영 피드백을 반영해 상세 확정 — 현 시점은 스펙 스케치.

## 위치

- 퍼널상 **리텐션 단계** — 발견(M1)·공급(M2)이 선행 ([PRD §3](../prd.md#3-핵심-가치와-퍼널)).
- 로그인 벽은 서비스 전체에서 **여기 한 곳** — 북마크/알림 설정 시점 ([PRD §4.6](../prd.md#46-인증과-로그인-벽)).
- **M3 완료 기준**: 북극성 지표(주간 재방문)를 유저 단위로 측정 가능해짐.

## 범위

1. 북마크 (인증 의존 — 구현 완료)
2. 마감임박 이메일 알림 (북마크 의존)
3. 로그인 혜택 — 내 제보 현황·승인 알림

---

## 0. 선행 — 인증 (구현 완료)

> 종전 이 문서의 §1. **이미 구현됐으므로 명세가 아니라 현황 요약**. 커밋 9597909, [SecurityConfig](../../src/main/kotlin/com/meepleday/user/SecurityConfig.kt).

- **OAuth2 소셜 로그인(Kakao·Google)**, 자체 비밀번호 미보유.
  - 기각 — 이메일·비밀번호 자체 인증: 비밀번호 저장·유출 책임·재설정 흐름 부담. 국내 대상엔 카카오 로그인이 가입 마찰 최저.
- `User`: id, provider(KAKAO/GOOGLE), providerId, email(**nullable**), displayName, role(USER/ADMIN), createdAt. `(provider, providerId)` unique. 최초 로그인 시 자동 프로비저닝.
- 전송 방식은 **세션 쿠키 + CSRF 쿠키**로 확정 — 종전 "세션 vs JWT" 열린 결정은 해소됨. 배포가 CloudFront 단일 도메인 경로 라우팅([ADR-0005](../adr/0005-deployment-target.md))이라 동일 오리진.
- `/api/admin/**` = `hasRole("ADMIN")`, `/api/me` = 인증, 그 외 공개. `/api/**` 인증 실패는 `401`(리다이렉트 아님).
- ADMIN 자가가입 불가 — `AdminAllowlistProperties` 허용목록으로 부여.
- 현재 유저는 `SecurityContext` 직접 접근 금지 → `CurrentUserProvider` 주입.

**남은 부채**
- **Kakao email 비보장**: 카카오 email 제공은 선택·심사 대상 → email을 식별자로 쓰지 말 것. 키는 `(provider, providerId)`, email은 nullable 부가정보(알림용).
- **계정 연결(linking)**: 같은 사람이 Kakao·Google로 각각 로그인하면 별도 User 2개. 통합은 복잡도 커서 defer.
- `Event.submitterName`/`submitterEmail`(텍스트)과 `submittedByUserId`(FK)가 **병존 중** → 익명 제보를 계속 허용하므로([PRD §4.6](../prd.md#46-인증과-로그인-벽)) 텍스트 필드 제거 여부는 M3에서 판단.

---

## 1. 북마크

**사용자 스토리**
- 로그인 사용자는 관심 이벤트를 북마크/해제, 내 북마크 목록 조회.

**엔티티**
- `Bookmark`: user_id, event_id, createdAt. `(user_id, event_id)` unique.

**엔드포인트(안)**
- `POST /api/events/{id}/bookmark`, `DELETE /api/events/{id}/bookmark`, `GET /api/me/bookmarks`.

**규칙**
- 북마크 목록도 이벤트 상태는 저장 안 하고 파생([ADR-0004](../adr/0004-derived-status.md)) — 목록 조회 시 N+1 주의(배치 로딩).
- 목록 화면(S7)은 피드와 동일한 **기간 그룹 타임라인** 레이아웃 재사용 ([ADR-0008](../adr/0008-timeline-layout.md)).
- 타 유저 자원 접근 차단 — 북마크 조회·삭제는 소유자 검증.

---

## 2. 마감임박 이메일 알림

**사용자 스토리**
- 북마크한 이벤트가 마감임박(ENDING_SOON, `endAt` 48h 이내) 진입 시 이메일 1회 수신.

**메커니즘**
- `@Scheduled` 배치(예: 시간당)로 ENDING_SOON 진입 이벤트 조회 → 해당 이벤트를 북마크한 User에게 발송.
- 상태는 파생이므로 배치는 시간 술어로 쿼리([ADR-0004](../adr/0004-derived-status.md)) — 별도 상태 컬럼 동기화 불필요.
- **`endAt`이 없는 이벤트(예고·상시)는 알림 대상에서 자연 제외** — 마감 개념이 없음.

**엔티티**
- `NotificationLog`: user_id, event_id, sentAt. `(user_id, event_id)` unique → **중복 발송 방지 게이트**(재실행·재시작에도 1회 보장).

**규칙**
- N+1 금지: 대상 이벤트·북마커·기발송 로그를 배치 조회 후 `groupBy`/`associateBy`로 조합.
- 발송 실패는 로그 미기록 → 다음 배치에서 재시도(멱등).

**카카오 email 비보장 대응 (확정)**
- email 없는 유저는 이메일 알림 대상에서 제외.
- **알림을 켜는 시점에 email이 없으면 그 자리에서 입력받음** — 로그인 시점이 아니라 알림 설정 시점. 마찰을 필요한 순간으로 미룸 ([PRD §4.7](../prd.md#47-추적-북마크알림)).
- email 없이도 북마크 자체는 동작(목록 관리 용도).

**기각한 대안** ([PRD §4.7](../prd.md#47-추적-북마크알림))
- **웹푸시**: email 불필요·무료지만 iOS Safari가 PWA 설치를 요구해 커버리지에 구멍.
- **키워드/조건 구독**: 리텐션 훅은 가장 강하나 초기 이벤트 밀도에선 조건이 무의미하게 넓거나 좁음.
- **ICS 캘린더 연동**: 구현이 싸고 "일정 서비스" 정체성에 맞지만, 알림을 외부 캘린더에 위임하면 **재방문이 발생하지 않아** 북극성 지표와 충돌.

---

## 3. 로그인 혜택 (제보자 귀속)

- 로그인 상태로 제보하면 `submittedByUserId`가 채워짐(이미 구현된 필드).
- 제공 혜택: 내 제보의 검수 상태 조회, 승인 시 알림, 기여 기록.
- 목적: 제보를 익명 허용하면서도 로그인 유인을 만듦 — 데이터 공급과 가입 전환을 동시에.

---

## 열린 결정 (착수 시 확정)

- **이메일 발송 프로바이더**: Resend(개발 친화) vs AWS SES(저비용·확장, [ADR-0005](../adr/0005-deployment-target.md) AWS 스택과 정렬). 초기 Resend 권장, 물량 증가 시 SES 재검토.
- **알림 주기·묶음**: 이벤트당 즉시 1회 vs 일 1회 다이제스트. 북마크 수가 적은 초기엔 즉시 1회로 충분할 전망.
- `submitterName`/`submitterEmail` 텍스트 필드 제거 여부.

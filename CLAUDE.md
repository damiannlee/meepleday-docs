# MeepleDay — 공통 지침

> [meepleday-backend](https://github.com/damiannlee/meepleday-backend), [meepleday-frontend](https://github.com/damiannlee/meepleday-frontend) 양쪽에 `docs/` git submodule로 포함되어, 각 레포 루트 `CLAUDE.md`에서 `@docs/CLAUDE.md`로 import된다. 레포별 특화 컨벤션은 각 레포 `CLAUDE.md` 참조.

## 제품

흩어져 있는 보드게임 이벤트 일정(펀딩·선주문·특가·오프라인 행사·예고)을 국내·해외 통합해 한 곳에서 보는 서비스. 제품 정의 단일 소스 = [prd.md](prd.md), 설계 근거 = [adr](adr).

## 현재 위치

- **PRD 확정, M0(재설계 기반) 완료, M1(발견) 착수 전.** 정체성을 "펀딩 애그리게이터" → "보드게임 이벤트 캘린더"로 재정의, Phase 1/2 구분 폐기.
- 로드맵: **M0 재설계 기반 → M1 발견 → M2 공급 → M3 추적 → M4+ 장기 비전** ([PRD §7](prd.md#7-로드맵-전면-재수립)).
- 구현 완료: 피드·상세·제보·검수, 남용 방지, 인증(Kakao/Google OAuth2, 세션 쿠키+CSRF), M0 데이터 모델(`Game` 엔티티, `OFFLINE_EVENT`, `ANNOUNCED` 상태).
- 미구현 주요 항목: URL 자동 채움, OG 프리렌더, 타임라인 레이아웃, 키워드 검색 UI, 게임 매칭 후보 제시 UI, 북마크·알림.

## 아키텍처 요점 (재서술 대신 포인터)

- 제보는 별도 테이블 없이 `Event.moderationStatus`로 통합 — [ADR-0002](adr/0002-moderation-on-event.md).
- 이벤트 진행 상태(ANNOUNCED/UPCOMING/ONGOING/ENDING_SOON/ENDED)는 저장 안 하고 파생 — [ADR-0004](adr/0004-derived-status.md). `startAt`/`endAt` 둘 다 null이면 `ANNOUNCED`.
- 게임 단위 묶음은 `Game` 1:N `Event`(`Event.gameId`, nullable FK) — [ADR-0007](adr/0007-game-entity.md). 동일 게임 판정은 사람이(검수 단계), 매칭 후보 UI는 M2.
- 첫 화면은 기간 그룹 타임라인 — [ADR-0008](adr/0008-timeline-layout.md). 현재 카드 그리드 구현은 교체 대상.
- 공유 유입이 주 채널 → CSR SPA 유지하되 **OG 프리렌더 필수**(미구현) — [ADR-0001](adr/0001-tech-stack.md).
- URL 자동 채움(단일 지시형 페치)은 자동 수집(대량 크롤링)과 별개 사안 — [ADR-0009](adr/0009-fetch-vs-crawl.md). SSRF·자원 상한·rate limit 필수.
- 검색은 `LIKE` 부분일치로 시작, 게임 제목은 `gameId` 집합 경유 **2단계 질의**(join 불가) — [ADR-0011](adr/0011-search.md), [spec/search.md](spec/search.md).
- 배포 타깃 AWS 서울(S3+CloudFront·EC2·RDS·Route53), 공개 배포는 M3 완료 후 — [ADR-0005](adr/0005-deployment-target.md).
- 익명 제보 남용: 검수 게이트가 1차 차단, honeypot+IP rate limit 최소 방어 — [ADR-0006](adr/0006-abuse-prevention.md).

## 공통 코딩 원칙 (언어 무관)

- 주석·로그 = 영어. 문서·커밋 메시지 = 한국어(개조식).
- wildcard import 금지. 매직넘버·반복 문자열은 상수화.
- 함수 바디 약 30줄을 기본 상한으로 — 넘으면 책임을 분리한 헬퍼로 추출.

## Git

- `main`(각 레포 기본 브랜치) 직접 커밋 금지 — 브랜치 생성 후 작업.
- 커밋 = Conventional Commits + 본문 한국어 개조식. PR 생성은 명시적 요청 시에만.

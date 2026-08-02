# Meepleon — 공통 지침

> [meepleon-backend](https://github.com/damiannlee/meepleon-backend), [meepleday-frontend](https://github.com/damiannlee/meepleday-frontend) 양쪽에 `docs/` git submodule로 포함되어, 각 레포 루트 `CLAUDE.md`에서 `@docs/CLAUDE.md`로 import된다. 레포별 특화 컨벤션은 각 레포 `CLAUDE.md` 참조.

## 제품

흩어져 있는 보드게임 이벤트 일정(펀딩·선주문·특가·오프라인 행사·예고)을 국내·해외 통합해 한 곳에서 보는 서비스. 제품 정의 단일 소스 = [prd.md](prd.md), 설계 근거 = [adr](adr).

## 현재 위치

최신 구현 현황·로드맵 위치는 [status.md](status.md) — CLAUDE.md 본문과 분리해 백엔드/프론트엔드 세션이 직접 갱신 가능.

## 크로스 레포 협업

- 백엔드·프론트는 서로의 레포를 직접 보지 않고 이 `docs`만을 인터페이스로 삼는다 — 작업 참고 자료도, 완료 내역 전달도 전부 여기로 통일.
- 작업 단위(PR) 완료 시 해당 레포는 PR과 함께 다음을 갱신한다:
  - 백엔드는 API 변경 시 [`docs/openapi.yaml`](openapi.yaml)을 재생성(gradle task, [백엔드 CLAUDE.md](https://github.com/damiannlee/meepleon-backend/blob/main/CLAUDE.md) 참조) — 요청/응답 스키마의 단일 소스.
  - 관련 `spec/*.md`는 요청/응답 스키마를 수기 전사하지 말고 `openapi.yaml`을 링크 + 행동 규칙(엣지케이스·검증·비즈니스 규칙)만 기술(해당 스펙이 없으면 신설).
  - 설계 결정이 있었다면 신규 ADR 또는 기존 ADR에 `## 개정 (YYYY-MM-DD)` 절 추가.
  - [`status.md`](status.md)의 구현 완료/미구현 목록 갱신.
- PR 자체의 경위·구현 디테일은 docs에 재서술하지 않는다 — 필요하면 PR 번호로만 포인터.
- 반대쪽 레포는 갱신된 docs를 참고해 작업을 시작한다 — 실제 착수는 여전히 "docs 참고해서 진행해" 사용자 지시가 트리거. 다만 백엔드 PR이 `docs/`를 건드리면 CI가 frontend 레포에 이슈를 자동 생성해 대기 목록으로 남긴다([백엔드 CI](https://github.com/damiannlee/meepleon-backend/blob/main/.github/workflows/notify-frontend.yml)) — 무엇을 아직 안 시켰는지 사람이 기억할 필요 없음.

## 아키텍처 요점 (재서술 대신 포인터)

- 제보는 별도 테이블 없이 `Event.moderationStatus`로 통합 — [ADR-0002](adr/0002-moderation-on-event.md).
- 이벤트 진행 상태(ANNOUNCED/UPCOMING/ONGOING/ENDING_SOON/ENDED)는 저장 안 하고 파생 — [ADR-0004](adr/0004-derived-status.md). `startAt`/`endAt` 둘 다 null이면 `ANNOUNCED`.
- 게임 단위 묶음은 `Game` 1:N `Event`(`Event.gameId`, nullable FK) — [ADR-0007](adr/0007-game-entity.md). 동일 게임 판정은 사람이(검수 단계), 매칭 후보 UI는 M2.
- 추적은 **북마크(`Event`) + 관심 게임 팔로우(`Game`) 병존** — [ADR-0013](adr/0013-bookmark-vs-game-follow.md). 팔로우는 신규 일정 등록 알림까지만이고 자동 북마크하지 않음. 행동 규칙은 [spec/tracking-model.md](spec/tracking-model.md).
- 생애주기 마일스톤은 `EventType.MILESTONE`(점 이벤트, `gameId` 필수) — [ADR-0014](adr/0014-milestone-event-type.md). 게임 페이지 외 전면 비노출(피드·필터·검색·OG), 북마크 불가, 파생 상태 대상 아님.
- 예고(`ANNOUNCED`)가 확정되면 **같은 row 갱신**(새 row 금지) — [ADR-0004 개정](adr/0004-derived-status.md). 전이 자체가 시작 알림 트리거.
- 첫 화면은 기간 그룹 타임라인이 기본값, 간트·카드그리드는 보조 토글 — [ADR-0008](adr/0008-timeline-layout.md)(2026-07-26 개정).
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
- 커밋 = Conventional Commits + 본문 한국어 개조식.

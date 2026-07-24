# Architecture Decision Records

의미 있는 설계 결정과 **기각한 대안**을 기록. 신규 결정은 번호를 이어 추가.

기존 ADR의 결정이 바뀌면 원문을 고치지 않고 **`## 개정 (YYYY-MM-DD)` 절을 덧붙인다** — 결정 이력 자체가 기록 가치이므로.

| # | 결정 | 상태 |
|---|---|---|
| [0001](0001-tech-stack.md) | 기술 스택 (Kotlin/Spring + React SPA) | Accepted · 개정 2026-07-23 (OG 프리렌더 조건 부가) |
| [0002](0002-moderation-on-event.md) | 제보를 별도 테이블 없이 Event 상태로 통합 | Accepted |
| [0003](0003-auth-last.md) | 인증을 마지막 개발 단계로 지연 | Accepted · 개정 2026-07-23 (구현 완료, 제보는 익명 유지) |
| [0004](0004-derived-status.md) | 이벤트 진행 상태를 저장하지 않고 파생 | Accepted · 개정 2026-07-23 (`ANNOUNCED` 분기 추가) |
| [0005](0005-deployment-target.md) | 배포 타깃 (AWS 서울, M3 이후 완성형 출시) | Accepted |
| [0006](0006-abuse-prevention.md) | 익명 제보 남용 방지 (honeypot + IP rate limit) | Accepted · 개정 2026-07-23 (URL 자동 채움 SSRF 방어 추가) |
| [0007](0007-game-entity.md) | 게임을 별도 엔티티로 두고 Event를 1:N 연결 | Accepted |
| [0008](0008-timeline-layout.md) | 첫 화면을 기간 그룹 타임라인으로 | Accepted |
| [0009](0009-fetch-vs-crawl.md) | URL 자동 채움을 자동 수집과 분리해 취급 | Accepted |
| [0010](0010-growth-target-unbuilt.md) | 성장 목표는 명시하되 구현하지 않는다 | Accepted |
| [0011](0011-search.md) | 키워드 검색 (LIKE 부분일치로 시작, FTS defer) | Accepted |

제품 정의는 [../prd.md](../prd.md), 기능명세는 검색 [../spec/search.md](../spec/search.md) · M3 [../spec/m3-tracking.md](../spec/m3-tracking.md), 성공지표·데이터 수급은 [../product.md](../product.md).

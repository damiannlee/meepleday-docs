# Architecture Decision Records

의미 있는 설계 결정과 **기각한 대안**을 기록. 신규 결정은 번호를 이어 추가.

| # | 결정 | 상태 |
|---|---|---|
| [0001](0001-tech-stack.md) | 기술 스택 (Kotlin/Spring + React SPA) | Accepted |
| [0002](0002-moderation-on-event.md) | 제보를 별도 테이블 없이 Event 상태로 통합 | Accepted |
| [0003](0003-auth-last.md) | 인증을 마지막 개발 단계로 지연 | Accepted |
| [0004](0004-derived-status.md) | 이벤트 진행 상태를 저장하지 않고 파생 | Accepted |
| [0005](0005-deployment-target.md) | 배포 타깃 (AWS 서울, Phase 2 이후 완성형 출시) | Accepted |
| [0006](0006-abuse-prevention.md) | 익명 제보 남용 방지 (honeypot + IP rate limit) | Accepted |

Phase 2 기능명세는 [../spec/phase2.md](../spec/phase2.md), 성공지표·데이터 수급은 [../product.md](../product.md).

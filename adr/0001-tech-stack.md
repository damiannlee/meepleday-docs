# 0001. 기술 스택

- 상태: Accepted
- 맥락: 국내+해외 보드게임 이벤트를 여러 사용자에게 공개하는 서비스. "한눈에 보기" UX가 핵심 가치.

## 결정

- Backend: **Kotlin + Spring Boot 3.4 + JPA + Flyway**
- DB: dev **H2**(PostgreSQL 호환 모드) / prod **PostgreSQL**
- Frontend: **React 18 + TypeScript + Vite** (SPA), 백엔드는 순수 REST

## 근거

- 주 스택(Kotlin/Spring) 역량 강화·포트폴리오 목적에 부합.
- 필터·정렬 중심의 인터랙티브 대시보드 UX → SSR보다 SPA가 유리.
- 프론트/백 분리로 포트폴리오 폭 확대, 추후 모바일 클라이언트 확장 여지.
- H2를 PostgreSQL 호환 모드로 써서 dev/prod 간 마이그레이션 이식성 확보 → Flyway 스크립트 하나로 양쪽 커버.

## 기각한 대안

- **Thymeleaf SSR 단일 앱**: 구현은 단순하나 복잡한 필터·상태 배지·달성률 바 등 인터랙션 구현이 번거롭고 UX 한계. 핵심 가치(한눈에 보기)와 상충.
- **dev/prod 동일 PostgreSQL(Testcontainers)**: 이식성·현실성은 최상이나 초기 스캐폴드 단계엔 셋업 비용 과함. 회귀 위험이 관측되면 재검토.

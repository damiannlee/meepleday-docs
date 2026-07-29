# 0012. 서비스명 "미플데이"(Meepleday) → "미플온"(Meepleon) 리브랜딩

- 상태: Accepted
- 맥락: 서비스명 "미플데이"(Meepleday)의 브랜드 어감·발음을 재검토 — "미플온"(Meepleon)이 meeple+on 구조로 더 잘 읽히고 어감이 낫다고 판단해 후보로 검토. 실행 전 도메인·상표 리스크를 먼저 확인.

## 결정

- **서비스명을 "미플온"(영문 표기 **Meepleon**, meeplon 아님)으로 확정.**
- 문서(PRD, ADR, 루트 CLAUDE.md 등) 내 "미플데이"/"Meepleday" 표기를 "미플온"/"Meepleon"으로 전면 치환.
- **GitHub 저장소명(`meepleday-backend`, `meepleday-frontend`)과 백엔드 Kotlin 패키지(`com.meepleday.*`)는 이번 변경 범위에서 제외** — 저장소 rename과 패키지 리팩터링은 각 레포에서 별도로 판단·실행할 사안으로 분리.

## 근거

- **어감/발음**: "미플온"이 "meeple + on"으로 분해돼 읽히는 반면, 대안 표기 "meeplon"은 축약된 형태라 원어민·비원어민 모두에게 어원이 덜 직관적으로 읽힘 → 영문 표기를 meepleon으로 확정.
- **리스크 사전 검증 완료**(이전 세션): 도메인 `meepleon.com`/`.io`/`.net`/`.app` 전부 미등록(구매 가능) 확인, KIPRIS 상표 검색에서 "미플온"/"MEEPLEON"/"MEEPLON" 정확 명칭 및 "미플"/"MEEPLE"/"온보드" 부분일치 전부 등록·출원 이력 0건 확인 — 상표·도메인 리스크는 이번 리브랜딩의 계기가 아니라 실행 전 확인한 전제 조건.

## 기각한 대안

- **"미플온" 대신 "미플런"("미플" + "런/run") 등 다른 후보**: 별도 세션에서 후보군을 검토했으나 상세 비교 근거는 기록되지 않음 — 최종적으로 어감·발음 기준 "미플온" 채택.
- **저장소명·패키지명까지 이번에 함께 변경**: 저장소 rename은 클론된 로컬 사본·CI·webhook 등 외부 참조가 걸린 되돌리기 어려운 작업이고, 패키지 리팩터링은 백엔드 코드 전반에 영향을 주는 별도 규모의 작업 — docs 리브랜딩과 범위를 분리해 각 레포에서 독립적으로 판단하도록 함.

## 트레이드오프

- 문서상 브랜드명과 실제 저장소 URL·패키지 경로(`meepleday-backend`, `com.meepleday.*`)가 당분간 불일치 — 코드를 처음 보는 사람에게는 혼란 요소가 될 수 있음. 저장소·패키지명을 언제 맞출지는 각 레포 팀(백엔드/프론트) 판단에 맡김.
- 도메인 미등록 확인은 스냅샷 시점 기준 — 실제 구매·등기는 별도 실행 필요, 이 ADR은 확인 결과만 기록.

## 개정 (2026-07-29)

- **저장소명·백엔드 패키지명 동기화 실행.** M1 착수 전·공개 배포 전([ADR-0005](0005-deployment-target.md))이라 외부 참조가 거의 없는 지금이 가장 싼 시점이라 판단, 트레이드오프에서 미뤄뒀던 항목을 처리.
  - GitHub 저장소: `meepleday-backend` → **`meepleon-backend`**, `meepleday-docs` → **`meepleon-docs`**. `meepleday-frontend`는 프론트 레포 팀이 별도 판단(이번 변경 범위 밖).
  - 백엔드 Kotlin 패키지: `com.meepleday.*` → **`com.meepleon.*`** 전면 치환.
- frontend 레포의 `docs` 서브모듈 URL은 여전히 구 `meepleday-docs`를 가리킴 — frontend 쪽에서 `.gitmodules` 갱신 필요(별도 안내 요망, 이 세션에서는 frontend 레포 접근 불가).

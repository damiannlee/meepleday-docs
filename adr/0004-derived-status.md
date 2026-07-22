# 0004. 이벤트 진행 상태를 저장하지 않고 파생

- 상태: Accepted
- 맥락: 이벤트는 예정/진행중/마감임박/마감(UPCOMING/ONGOING/ENDING_SOON/ENDED) 상태를 가짐. 이 상태는 시간이 지나면 저절로 바뀜.

## 결정

- `EventStatus`를 DB 컬럼으로 저장하지 않고 `startAt`/`endAt`과 현재 시각으로 **읽는 시점에 계산**([EventStatus.of](../../src/main/kotlin/com/meepleday/event/EventEnums.kt)).
- `ENDING_SOON`은 ONGOING 중 `endAt`이 임계값(48h) 이내인 파생 상태.
- 상태 기반 필터는 시간 조건 술어로 변환해 쿼리([EventSpecifications.status](../../src/main/kotlin/com/meepleday/event/EventSpecifications.kt)).
- 현재 시각은 `Clock` 빈 주입으로 테스트 가능하게.

## 근거

- 저장하면 배치/스케줄러로 끊임없이 동기화해야 하고, 갱신 누락 시 상태가 실제와 어긋남.
- 파생은 항상 정확하고 유지보수 지점이 없음.

## 기각한 대안

- **상태 컬럼 저장 + 스케줄러 갱신**: 조회는 단순해지나 동기화 부담·불일치 위험. 데이터 규모가 커져 계산 비용이 관측되기 전엔 불필요한 복잡도.

## 트레이드오프

- 상태로 정렬/필터할 때 시간 술어가 쿼리에 들어감 → 인덱스(`moderation_status, end_at`)로 커버.
- JPA Criteria는 `NULLS LAST` 미지원 → Hibernate `order_by.default_null_ordering=last` 전역 설정으로 마감일 없는 상시 이벤트를 뒤로 정렬.

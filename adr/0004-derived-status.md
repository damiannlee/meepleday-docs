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

## 개정 (2026-07-23) — 예고 이벤트를 위한 `ANNOUNCED` 분기 추가

- 맥락: [PRD](../prd.md)에서 **일정만 발표된 예고 이벤트**를 수용하기로 결정 ([PRD §5.2](../prd.md#52-예고-섹션-분리)).
- **파생 결정은 유지.** 저장하지 않고 계산한다는 원칙은 그대로 옳음.
- **결함**: `startAt`/`endAt`은 이미 nullable이지만, `EventStatus.of()`가 **둘 다 null이면 `ONGOING`으로 떨어짐**. 날짜 미정 예고 이벤트가 "진행중"으로 표시되는 오동작.
  ```
  if (endAt != null && endAt.isBefore(now)) return ENDED
  if (startAt != null && startAt.isAfter(now)) return UPCOMING
  if (endAt != null && ...) return ENDING_SOON
  return ONGOING   // ← startAt·endAt 둘 다 null인 예고가 여기로 떨어짐
  ```
- **개정 내용**
  - `EventStatus`에 **`ANNOUNCED`** 추가 — `startAt`·`endAt`이 모두 null인 상태(= 일정 미확정 예고).
  - `Event.scheduleNote`(자유 입력, 예: `"2026년 4분기 예정"`)를 신설해 예고의 시기를 사람이 읽을 수 있게 표기. **정밀도 열거형은 만들지 않음** — 예고가 소수인 단계에선 과설계, 대신 예고 섹션 내부 정렬은 포기.
  - 타임라인 본문에는 날짜 확정 이벤트만, `ANNOUNCED`는 하단 예고 섹션으로 분리 ([ADR-0008](0008-timeline-layout.md)).
  - `endAt`이 없으면 `ENDING_SOON`에 진입할 수 없으므로 마감임박 알림 대상에서 자연 제외.
- 기각 — **`datePrecision` 열거형(연/분기/월/일)**: 표현·정렬 모두 정확해지지만 예고 건수가 적은 현 단계엔 과설계. 예고가 늘면 재검토.
- 기각 — **예고를 별도 엔티티로 분리**: `Event`와 스키마가 사실상 같아 중복. [ADR-0002](0002-moderation-on-event.md)에서 제보를 통합한 것과 같은 논리.

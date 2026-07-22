# 0002. 제보를 별도 테이블 없이 Event 상태로 통합

- 상태: Accepted
- 맥락: 크롤러 구현 전까지 데이터 공급은 **운영자 직접 등록 + 사용자 제보(크라우드소싱)** 두 갈래. 제보는 검수(승인/반려)를 거쳐 게시.

## 결정

- 제보용 별도 엔티티를 두지 않고 `Event`에 `moderationStatus`(PENDING/PUBLISHED/REJECTED)와 `submittedBy`(현재는 `submitterName`/`submitterEmail` 텍스트) 필드로 통합.
- 운영자 직접 등록은 곧장 PUBLISHED, 사용자 제보는 PENDING으로 생성.
- 공개 피드/상세는 PUBLISHED만 노출, 검수 큐는 PENDING 조회.

## 근거

- 승인 시 별도 테이블 → Event 복제 로직 불필요. 단일 테이블·단일 소스.
- 제보와 정식 이벤트의 스키마가 사실상 동일 → 중복 스키마 제거(YAGNI).
- 상태 전이(publish/reject)를 Event 도메인 메서드로 캡슐화해 응집도 확보.

## 기각한 대안

- **`Submission` 별도 테이블 + 승인 시 Event 생성**: 원본 제보 감사 추적은 깔끔하나, 이중 스키마·복제 로직·두 테이블 동기화 부담. 초기 스코프엔 과함.
- 감사 추적이 실제로 필요해지면(악성 제보 대응 등) `submission_log` 같은 append-only 이력 테이블을 별도로 도입해 해결 — 정식 데이터 모델을 복잡하게 만들지 않음.

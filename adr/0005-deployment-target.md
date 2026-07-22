# 0005. 배포 타깃 — AWS 서울

- 상태: Accepted
- 맥락: 무료 티어 소진 → 유료 전제. 목표①(취업 포트폴리오)이 최우선이라 **채용 시장 키워드 가치가 큰 AWS** 채택. 관객이 한국인 → **서울 리전(ap-northeast-2)**. **공개 배포는 Phase 2 완료 후 완성형으로 출시**(근거 [ADR-0003](0003-auth-last.md)) → **첫 배포 시점에 이미 계정 등 유실-불가 데이터가 존재.** 워크로드는 저트래픽·읽기 위주(트래픽 모델 [product.md](../product.md)).

## 결정

- **클라우드 = AWS 서울 리전.** SPA·API·DB·DNS·CDN을 AWS 단일 스택으로 통일.
- 구성:
  - Frontend(정적 SPA): **S3 + CloudFront**.
  - Backend(Spring Boot): **EC2 단일 인스턴스**(Graviton t4g) + Docker.
  - DB: **관리형 RDS(Postgres, 단일 AZ)** 를 출시부터.
  - CDN: CloudFront가 SPA와 `/api`(읽기 캐시)를 **동일 도메인 경로 라우팅** → 오리진 부하·CORS 완화.
  - DNS: Route53.
  - **ALB·NAT Gateway 미사용**(비용). 퍼블릭 서브넷 + CloudFront→오리진.

## 근거

- **포트폴리오①**: `EC2·RDS·CloudFront·S3·Route53`가 이력서에 직접 박히는 시장성 키워드. Fly.io·VPS보다 채용 인지도 높음.
- 저트래픽엔 **용량이 제약이 아님** → 최소 구성으로 비용 억제. 오토스케일·멀티AZ는 이 규모에 오버킬.
- **RDS를 출시부터 쓰는 이유**: 공개 배포가 Phase 2 이후라([ADR-0003](0003-auth-last.md)) 첫 배포 시점에 이미 유실-불가 계정 데이터가 존재 → 저비용 콜로케이트 PG로 시작할 "유실 감내 구간"이 배포상 없음. 관리형 RDS로 직행해 자동 백업·복구를 처음부터 확보.
- 서울 리전으로 KR 지연 최소(~5–15ms).

## 예상 비용 (서울, 월, 근사)

| 항목 | 온디맨드 | 1yr Savings Plan |
|---|---|---|
| EC2 t4g.small + EBS | ~$18 | ~$12 |
| RDS(Postgres micro, single-AZ, 20GB) | ~$20 | ~$14 |
| CloudFront + S3 + Route53 | ~$3 | ~$3 |
| **합계** | **~$41** | **~$28** |

- egress는 무료 범위(월 100GB / CloudFront 1TB)라 사실상 $0. 이미지는 원본 URL 참조라 미서빙.

## 기각한 대안

- **셀프호스팅 VPS(도쿄) + Docker Compose**: 최저가($8–14)·깊은 제어지만 취업 키워드 약하고 서울 리전 없음(도쿄까지). 포트폴리오①에 밀림.
- **관리형 PaaS(Fly.io/Railway)**: 운영 편하나 포트폴리오 가치 얕음. Railway는 KR 리전 부재로 지연 불리.
- **AWS 정석 풀스택(Fargate + ALB + Multi-AZ RDS + NAT)**: 신뢰성·확장성 최상이나 월 $60–110로 이 트래픽엔 과설계·낭비. 트래픽이 관측되면 재검토.
- **단계적 콜로케이트 PG(A)→RDS(B)**: 초기 RDS 비용 이연이 목적이었으나, 공개 배포가 Phase 2 이후라 유실-감내 배포 구간이 없어 A의 적용 창이 소멸 → 폐기, RDS 직행.
- (이전 초안의 Cloudflare Pages + Fly.io + Neon 조합 폐기.)

## 트레이드오프 / 후속 부채

- **요금 폭탄 방지**: NAT Gateway(월 $32)·유휴 EIP·미사용 EBS 스냅샷·Multi-AZ 실수 켜짐 — 4대 조용한 과금원 금지. **AWS Budgets 경보 첫날 설정(필수).**
- 백엔드 **Dockerfile 미작성** → 후속 구현 과제. 이미지는 ECR 푸시.
- CloudFront ↔ 오리진 HTTPS·오리진 보호(커스텀 헤더 등) 구성 필요.
- **배포 리스크 집중**: 개발 내내 로컬 실행(무배포) → 실제 AWS 환경 검증이 Phase 2 후 첫 배포에 몰림. 완화 위해 출시 전 스테이징성 리허설 1회 권장.

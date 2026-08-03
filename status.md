# 구현 현황

> 제품 정의는 [prd.md](prd.md), 설계 근거는 [adr](adr) 참조. 이 파일은 "지금 어디까지 됐는지"만 추적 — 백엔드/프론트엔드 PR 완료 시 함께 갱신.

- **PRD 확정, M0(재설계 기반) 완료, M1(발견) 착수 전.** 정체성을 "펀딩 애그리게이터" → "보드게임 이벤트 캘린더"로 재정의, Phase 1/2 구분 폐기.
- 로드맵: **M0 재설계 기반 → M1 발견 → M2 공급 → M3 추적 → M4+ 장기 비전** ([PRD §7](prd.md#7-로드맵-전면-재수립)).
- 구현 완료: 피드·상세·제보·검수, 남용 방지, 인증(Kakao/Google OAuth2, 세션 쿠키+CSRF), M0 데이터 모델(`Game` 엔티티, `OFFLINE_EVENT`, `ANNOUNCED` 상태), URL 자동 채움(`POST /api/events/prefill`, OG 메타 기반 — [ADR-0009](adr/0009-fetch-vs-crawl.md)), **M3 북마크**(`POST`/`DELETE /api/events/{id}/bookmark`, `GET /api/me/bookmarks`, `GET /api/me/submissions` — [spec/tracking-model.md §1](spec/tracking-model.md#1-북마크)).
- 미구현 주요 항목: OG 프리렌더, 타임라인 레이아웃(간트/그룹/카드그리드 토글), 키워드 검색 UI, 게임 매칭 후보 제시 UI. 마감 임박 알림은 M3 범위에서 빠져 장기비전으로 이동 — [PRD §8.5](prd.md#85-마감-임박-알림).
- **2026-08-02 추적 모델 확정(기획만, 구현 없음)**: 북마크(일정) + 관심 게임 팔로우(게임) 병존, `EventType.MILESTONE` 신설, 예고→확정 시 같은 row 갱신 — [ADR-0013](adr/0013-bookmark-vs-game-follow.md) · [ADR-0014](adr/0014-milestone-event-type.md) · [spec/tracking-model.md](spec/tracking-model.md). **M3는 북마크까지만**, 팔로우·마일스톤은 게임당 평균 연결 이벤트 2건 이상 관측 후.

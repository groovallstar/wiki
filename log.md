# Log

인제스트·린트 이력의 시간순 append-only 로그.

## [2026-04-30] ingest | Ubiquitous Language (DDD)
- `sources/dev/evans-2003-ddd.md` 신규 (Evans 2003 DDD 원서 요약).
- `sources/dev/pocock-2026-skills.md` 신규 (mattpocock/skills 카탈로그, `grill-with-docs` 4단계 정식화).
- `concepts/ubiquitous-language.md` 신규 (사용자·개발자 어휘 통일 = 번역 비용 0; Pocock 운용 절차 인용).
- `index.md` concepts/sources 카탈로그 갱신.
- 결정 근거: mattpocock/skills 검토 시 옵션 (b) 채택 — 카탈로그 비교 페이지(`agent-skills.md` 비대화) 회피, DDD 일반 도메인 지식 1페이지만 신설. NER 응용 예시는 프로젝트 독립성 원칙으로 제외. §출처는 schema §페이지 템플릿에 따라 `sources/` 포인터로만 표기, 외부 URL 은 source 파일 메타에만.

## [2026-04-27] lint | 수록 원칙·역참조 정합성 점검
- `concepts/agent-skills.md`: 호스트 저장소 파일명 1건 일반화, `docs/schema.md` → `schema.md` 경로 정정.
- `sources/env/python-editable-install-research.md`: 누락됐던 `## 인용하는 위키 페이지` 역참조 섹션 추가 (`concepts/src-layout-packaging.md`).
- `sources/dev/epril-2026-session-context-handoff.md`: 간접 참조였던 `concepts/token-management.md` 줄 삭제 (실제 직접 인용은 `concepts/session-handoff.md` 한 곳).
- 잔여 이슈: `concepts/agent-skills.md` 템플릿 구조(개요/핵심 원칙/...) 미준수는 정보 카테고리로 미수정 (콘텐츠 성격상 강제 불요).

## [2026-04-24] ingest | 세션 컨텍스트 핸드오프 4계층 전략
- `sources/dev/epril-2026-session-context-handoff.md` 신규 (정도현 2026-04-23 원문 요약).
- `concepts/session-handoff.md` 신규 (4계층 전략 · Handoff 문서 필수 섹션 · Registry 패턴 · 안티패턴).
- `concepts/token-management.md` §3, §4에 `concepts/session-handoff.md` cross-reference 추가 (focus 지시·hypothesis 원칙 위임).
- `index.md` concepts/sources 카탈로그 갱신.

## [2026-04-22] ingest | Cohen's kappa + Wikidata anchor verification
- `sources/labeling/cohen-1960-kappa.md` 신규 작성 (Cohen 1960 원문 요약; 이후 reorg 로 labeling/ 으로 이동)
- `sources/labeling/vrandecic-2014-wikidata.md` 신규 작성 (Wikidata CACM 2014 원문 요약; 이후 reorg 2건으로 labeling/ 으로 이동)
- `concepts/inter-annotator-agreement.md` 신규 작성 (κ 계열·NER 맥락·LLM silver 검증 통합)
- `concepts/kb-anchor-verification.md` 신규 작성 (KB 앵커 검증 방법론 통합)
- `index.md` 신규 생성 (전체 카탈로그)
- 양쪽 sources ↔ concepts 교차 참조 확립

## [2026-04-22] reorg | sources/ 주제별 하위 폴더 도입
- 초기 분류: dataset·env·dev 3종. `cohen`·`ramshaw-marcus` 는 루트 유지.
- 영향받은 concepts 5건 참조 경로 일괄 갱신.
- `index.md` 폴더 구조 반영.

## [2026-04-22] reorg | sources/ 분류 체계 재편 (ner·labeling 신설)
- `sources/dataset/` → `sources/ner/` 리네임.
- `ramshaw-marcus-1995-bio-tagging.md` 를 루트에서 `sources/ner/` 로 이동 (NER 스킴 원조).
- `sources/labeling/` 신설: `cohen-1960-kappa.md`·`vrandecic-2014-wikidata.md` 이동 (라벨링 품질·검증).
- 영향받은 concepts 3건 (bio-tagging·inter-annotator-agreement·kb-anchor-verification) 참조 경로 갱신.
- `index.md` 새 구조(ner·labeling·env·dev) 반영.

## [2026-04-22] schema | 폴더 정책 기록 (sources topic-grouped, concepts flat)
- `schema.md §디렉토리 구조` 에 현재 sources topic 폴더 구조 반영.
- 신규 하위 절 "폴더 정책: sources 는 topic-grouped, concepts 는 flat" 추가.
- concepts flat 유지 결정 근거와 재검토 임계치 명시 (페이지 수·클러스터 균질성·cross-ref 비율).

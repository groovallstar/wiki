# Log

인제스트·린트 이력의 시간순 append-only 로그.

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

# Log

인제스트·린트 이력의 시간순 append-only 로그.

## [2026-06-08] lint | dynamic-workflows ingest 직후 전수 점검
- 끊어진 링크 0 / 고아 0 / source 미인용 concept 0 / 모순·낡은 주장 0.
- 수록 원칙 검사 히트는 전부 오탐: `concepts/token-management.md`의 `.claude/settings.json`(Claude Code 일반 설정명, 2026-05-20 정정 시 수용), `concepts/src-layout-packaging.md`의 `/work/my-project`(일반 예시 경로). 호스트 저장소 누출 아님.

## [2026-06-08] ingest | Anthropic 2026 Dynamic Workflows (Claude Code)
- `concepts/dynamic-workflows.md` 신규 (작업별 하네스·서브에이전트 오케스트레이션·세 실패 모드[게으름·자기편애·목표 표류]·패턴 카탈로그 6종·비교>절대 검증 원리).
- `concepts/loop-verification-gate.md` 신규 (적대적 검증을 루프 완료 관문으로 구체화: 결정적 선통과→격리 반박자→판정 ground truth, 재진입 비-block·무한루프 차단·캐시 TTL·모델 tier 비용).
- `sources/dev/anthropic-2026-dynamic-workflows.md` 신규 (Anthropic 블로그 요약).
- 양방향 교차 참조: source ↔ 신규 concept 2건. 신규 concept → `agentic-data-generation`(verifier/judge 독립=self-preferential bias 대응 동형)·`kb-anchor-verification`(독립 검증) 발신. 역방향 보강: `agentic-data-generation`에 backlink 추가.
- `index.md` concepts/sources 카탈로그 갱신.
- 결정 근거: 적용한 검증 게이트는 프로젝트 독립 *방법론*으로만 수록(호스트 경로·스크립트·파일명 제외, 수록 원칙 §1). loop-verification-gate를 dynamic-workflows의 하위 구체화로 분리(개념 단위). dev/ topic 배치(개발 도구·에이전트 계열).

## [2026-06-05] query-filed | 서브에이전트마다 별도 LLM 모델이 필요한가
- `concepts/agentic-data-generation.md` §3에 "역할 ≠ 모델 수" 단락 추가. 핵심: 4개는 역할이지 모델 인스턴스가 아니며(원 실험은 orchestrator·challenger·judge 한 모델 공유, 4+역할을 2종으로 운영), 공유 정책은 역할별로 비대칭 — solver 쌍은 *상관*(격차=난이도 측정), verifier/judge 는 *독립*(자기 심판 회피). IAA·kb-anchor 로 발신 링크.

## [2026-06-05] lint | Autodata ingest 직후 전수 점검
- 끊어진 링크 0 / 고아 페이지 0 / source 미인용 concept 0 / 모순·낡은 주장 0.
- 수록 원칙: 검사 히트(`my_package`·`src/foo`·`__init__.py`·`PYTHONPATH=...`)는 전부 `concepts/src-layout-packaging.md`·`sources/env/python-editable-install-research.md`의 일반 예시 — 호스트 저장소(`src/ner`) 누출 아님, 오탐 처리.
- 수정 1건: `sources/labeling/kulikov-2026-autodata.md` "인용하는 위키 페이지"가 실제 인용 않는 concept 3개를 나열(직전 ingest 오류) → 직접 인용자(`agentic-data-generation.md`)만 남기고 나머지는 "관련 개념(직접 인용 안 함)" 하위 절로 분리. 관례(vrandecic·cohen=직접 인용자만) 정합.

## [2026-06-05] ingest | Kulikov 2026 Autodata (Agentic Self-Instruct)
- `concepts/agentic-data-generation.md` 신규 (데이터 과학자 루프 3단계·4역할 Agentic Self-Instruct·난이도 격차 수용 게이트·메타 최적화·보상 해킹 가드레일·기존 합성법 비교).
- `sources/labeling/kulikov-2026-autodata.md` 신규 (Meta AI RAM 블로그 요약).
- 양방향 교차 참조: source ↔ 신규 concept. 신규 concept → `kb-anchor-verification`(독립 심판 원리)·`data-splitting`(held-out·Goodhart·adaptive overfitting)·`inter-annotator-agreement`(자동 심판 편향)로 발신. 역방향 보강: `kb-anchor-verification` §8(생성 시점 검증 메모)·`data-splitting` §adaptive overfitting(합성 루프 동형) 에 backlink 추가.
- `index.md` concepts/sources 카탈로그 갱신.
- 결정 근거: 새 `data/` topic 대신 `sources/labeling/` 배치(데이터 품질·검증 계열, 신규 topic 임계치 ≥2건 미달). verifier-judge·격차 게이트는 프로젝트의 PII 주입+교차검증 증강과 직접 연결되는 재사용 방법론으로 판단해 concept 승격.

## [2026-05-20] lint | token-management `.claudeignore` 사실 정정
- `concepts/token-management.md` §1: `.claudeignore`(현행 Claude Code 미구현)를 작동 기제 `permissions.deny`(`.claude/settings.json`)로 정정. 원칙은 유지, 원문 주장과의 불일치를 검증 메모로 명시(모순 진술 회피). 웹 검증 근거: open issue #579/#29455/#30810/#36163.
- 잔여 이슈(사용자 보류): `concepts/agent-skills.md` 신규 3개(doubt-driven-development, git-workflow-and-versioning, interview-me) 미평가. 카탈로그 20→23 증가, 기존 20개는 개명·삭제 없음.

## [2026-05-20] ingest | 데이터 분리·평가 설계
- `concepts/data-splitting.md` 신규 (train/dev/test·train-dev 4분할·IID vs OOD·분포 시프트 3유형·temporal/walk-forward·adaptive overfitting 이론 vs 실증·지표 선택 연결).
- `sources/ner/` 에 핵심 6건 신규: ng-2018-ml-yearning, quinonero-candela-2009-dataset-shift, recht-2019-imagenet-generalize, dwork-2015-reusable-holdout, bergmeir-benitez-2012-timeseries-cv, koh-2021-wilds.
- 6 sources ↔ `concepts/data-splitting.md` 양방향 교차 참조 확립. `concepts/inter-annotator-agreement.md`(평가 천장 기준자)로 발신 링크.
- `index.md` concepts/sources 카탈로그 갱신.
- 결정 근거: 사용자 지정으로 방법론 핵심 6건만 ingest(NER 엔티티-중복 특화 3건 보류), 신규 `evaluation/` topic 대신 `sources/ner/` 배치. 통합 1페이지 입도 채택(데이터 분리 단일 주제). Recht(실증)와 Dwork(이론)의 adaptive overfitting 결론 차이를 본문에 병기. NER 특화 lexical-overlap 분할은 전용 출처 후속 ingest 대상으로 메모만.

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

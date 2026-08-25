# Log

인제스트·린트 이력의 시간순 append-only 로그.

## [2026-08-25] ingest | vLLM 운영 문서·엔진 로그 — 서버 운영 개념 신설
- `concepts/vllm-serving-operations.md` 신규: 기동 5단계와 각 단계의 로그, 메모리 예산 나눗셈(예산 → 가중치·임시 메모리 차감 → 토큰 환산 → 동시성 배수), 손잡이 넷의 상충, 운행 통계 줄 읽기, 지표 3조합으로 병목 가르기, 기동 실패 6유형.
- `sources/env/vllm-serving-operations-docs.md` 신규: 로그 형식 문자열 원문, 동시성 배수가 토큰 수보다 먼저 계산된다는 점, CUDA graph 회계 기본 활성, preemption 경고 원문과 대처 우선순위, 지표 이름·유형, 기동 오류 원문.
- **1차분과의 역할 분담**: `llm-inference-serving` 은 왜 빠르고 느린가(개념 축), 신설 페이지는 그것이 어떤 이름으로 화면에 나타나는가(운영 축). 1차분의 §서버를 띄울 때 만나는 손잡이들에서 신설 페이지로 포인터를 걸었다.
- **도구 종속 페이지를 concepts 에 둔 근거**: 로그 문자열을 인용하려면 엔진을 특정해야 한다. `schema.md` §수록 원칙은 호스트 저장소의 경로·스크립트·결과 파일명을 금지할 뿐 도구 옵션명을 금지하지 않고, 위키를 공유하는 다른 저장소에서도 같은 엔진을 쓰므로 재사용된다. 페이지 머리에 이름은 엔진 종속이고 구조는 아님을 명시했다.
- `index.md` 에 개념 1건·소스 1건 반영.

## [2026-08-20] ingest | vLLM 추론 서빙 문서 — LLM 서빙 성능 개념 신설
- `concepts/llm-inference-serving.md` 신규: prefill/decode 2단계를 축으로 KV 캐시·지연/처리량·양자화(`W4A16`·`group_size`·`ignore`)·MoE·attention 변형(full/sliding/linear/hybrid)·prefix caching·speculative decoding(MTP)을 정리.
- `sources/env/vllm-inference-serving-docs.md` 신규: prefix caching 블록 해시와 전체 블록 제약, speculative decoding 의 greedy 동일성 보장, MTP 권장 추측 토큰 1개와 고동시성 처리량 대가.
- `index.md` 에 개념 1건·소스 1건 반영.

## [2026-08-07] query-filed | 토큰 관리 페이지의 유효성 비판적 검토
- 질문: `concepts/token-management.md` 가 위키 취지에 맞고 아직 유효한가.
- **진단**: 수록 원칙은 글자 그대로는 안 어긴다(호스트 경로 없음·프로젝트 독립). 어긋나는 것은 위키가 말한 적 없는 가정이다 — 다른 개념 페이지는 논문·인지 원리에 기대 수십 년 단위로 유효한 반면, 이 페이지는 블로그 1건과 특정 제품의 슬래시 명령·설정 파일에 기대 릴리스 단위로 낡는다. `schema.md` 는 프로젝트 독립성만 요구하고 시간 독립성은 요구하지 않아 규칙으로는 안 걸린다.
- **§11 도구별 역할 분리 삭제**: 그 절의 유일한 근거는 "Claude 쪽 컨텍스트가 작다"였는데 그 제약이 사라졌다(현행 주요 모델은 1M 컨텍스트, claude-api 스킬로 확인). 제약이 없어지면 처방("코드베이스 맵핑은 긴 컨텍스트 도구에")은 근거 없이 떠 있는 지시문이 되고, 읽는 사람이 없는 제약을 피해 작업을 세 도구로 쪼갠다.
- **절감률 4건에 조건 표기**: 76.1%·20~30%·26%/+16.2%p·80% 이상은 전부 같은 블로그 1건에서 왔고 워크로드·측정 방법·비교 대상이 원문에 없다 → "저자 보고"로 표기하고 76.1% → ~76% 로 낮췄다(단일 사례인데 소수점이 정밀도를 과장). §1 의 30~40% 는 이미 검증 메모 안에서 원문 귀속돼 있어 그대로 뒀다. 이 처리는 `kwok-2026` preprint 수치를 "저자 보고"로 단 것과 같은 기준이다.
- **§8 중복 해소**: `session-handoff` §Tier 1 과 같은 격리 대상 목록을 표로 다시 쓰고 있었다 → §3·§4 와 같은 포인터 형태로 축약. **§10 보강**: 지연 로딩·도구 검색으로 서버를 연결한 채 상시 점유를 줄이는 길이 생겨, 끄는 것만이 수단이 아니게 됐다.
- **관측 시점 표기 신설**: 페이지 머리에 "도구 운용 항목은 2026-08 시점 기준"과 절감률 출처 한계를 명시. 소스 쪽(`jeong-2026`)에는 원문 목록을 그대로 두되 1번의 수단 대체와 11번의 전제 소멸을 검증 메모로 붙였다 — 소스는 사실 보고, 개념은 채택 판단이라는 기존 역할 분담을 따랐다.
- **보류**: 개념 페이지에 관측 시점 필드를 강제할지는 `schema.md` 를 안 건드렸다. 지금 해당하는 페이지가 `token-management`·`agent-skills` 둘뿐이라, 셋째가 생길 때 규칙으로 올린다(§라벨·스코프의 "가리킬 것이 생길 때 만든다"와 같은 원칙).

## [2026-08-07] lint | agent-skills 카탈로그 낡음 해소 + 운영문서 정합
- 전 40개 페이지 6축 점검. 구조축 전부 0건 — 끊어진 링크 0·고아 0·index 누락/dangling 0·concepts §출처 누락 0·교차참조 양방향 불일치 0. 수록 원칙 히트(`src/foo`·`my_package`·`tests/`·`uv sync`)는 전부 packaging 페이지의 일반 예시로 오탐, 호스트 저장소 누출 0건.
- **낡은 주장 해소 (2026-05-20 부터 이월, 사용자 보류였음)**: GitHub API 로 상류 재확인 — 실제 24종(WebFetch 는 24개를 나열하고 총계를 28로 답해 신뢰 불가, API 를 정본으로 삼음). 2026-04 관측 21종 대비 삭제 0·신규 3종. `sources/dev/addyosmani-agent-skills.md` 목록을 24종·"2026-08 시점"으로 갱신하고 증분 3종을 한 줄씩 풀이.
- `concepts/agent-skills.md` 신규 3종 평가 반영: `doubt-driven-development` → 강력추천 #4(원문 SKILL.md 대조; 위키가 이미 `loop-verification-gate`·`dynamic-workflows` 로 들고 있는 격리 반박자 원리의 개인 작업 단위 형태), `interview-me` → 보조 #10(착수 전 의도 좁히기), `observability-and-instrumentation` → 제외(배포·운영 계열, 기존 제외 기준과 동일). 보조 번호 4~8 → 5~9 재정렬.
- 같은 페이지의 **미분류 2건 정리**: `git-workflow-and-versioning`(2026-04 목록에 있었으나 한 번도 평가된 적 없음) → 제외 + 근거 명시(트렁크 기반·잦은 소단위 커밋이라는 스킬 기본값이 저장소 자체 규약과 경쟁). `documentation-and-adrs`(2026-06-12 에 삭제했으나 상류에는 실재) → 걷어낸 이유를 남겨 카탈로그 대사를 24/24 로 닫음.
- **모순 해소**: 제외 목록의 `context-engineering`(카탈로그 안 스킬 파일)과 `concepts/context-engineering.md`(핵심 개념으로 승격된 분야)가 같은 이름이라 상충으로 읽히던 것 → 가리키는 대상이 다름을 한 줄로 구분.
- **개념 그래프 고립 해소**: `agent-skills` 는 in=0 out=0 이라 index 로만 닿았음 → §관련 개념 신설(`loop-verification-gate`·`dynamic-workflows`), 양쪽에 역링크 추가.
- **운영문서 정합 3건**: `schema.md` 예시 블록이 addyosmani 소스를 reorg 이전의 sources 루트 경로로 적고 있던 것을 `sources/dev/addyosmani-agent-skills.md` 로 정정(2026-06-12 가 "무해, 미수정"으로 남긴 잔여 — 예시 블록은 다음 ingest 가 베끼는 자리라 틀린 형태를 가르친다). `index.md` 를 "(선택)"에서 상시 갱신 대상으로 표기 통일(30행 서술·docwiki 스킬과 어긋나 있었음, `log.md` 는 선택 유지). `README.md` 구조 블록에 `index.md`·`log.md`·`sources/<topic>/` 반영해 `schema.md` 와 동기화.
- **외부 URL 이관 2건**: `concepts/agent-skills.md`·`concepts/token-management.md` 본문 메타의 원문 URL 제거 — 2026-04-30 결정("외부 URL 은 source 파일 메타에만, concepts §출처는 `sources/` 포인터로만")과 어긋나 있었다. 두 URL 모두 각 source 파일에 이미 있어 정보 손실 없음.
- 잔여: sources 10건의 템플릿 필수 절 누락과 `concepts/agent-skills.md` 구조 미준수는 2026-04-27·2026-07-08 이 "글자강제하면 정보 손실"로 보류한 건이라 유지. flat `concepts/` 재검토 임계 근접(14/15).

## [2026-07-24] ingest | Kwok 2026 LLM-as-a-Verifier
- `concepts/verifier-score-resolution.md` 신규 (연속 점수=로짓 기댓값·세 해상도 축[값 G/통계 K/기준 C]·이산 판사 tie 실패 모드·명목 κ↔연속 α 이동·로짓 접근 전제).
- `sources/labeling/kwok-2026-llm-as-verifier.md` 신규 (arXiv preprint 요약; 원문 HTML 2회 독립 대조로 Eq.3.1·헤드라인 4수치·tie 26.7%@k=1·한계 3종 확정).
- 양방향 교차 참조: source ↔ 신규 concept. 신규 concept → `loop-verification-gate`(직교 축: 누가/어디서 vs 얼마나 세밀)·`inter-annotator-agreement`(연속화 시 κ→α/ICC)·`agentic-data-generation`(judge 신호 해상도)로 발신. 역방향 보강: 세 페이지 각각에 backlink 추가.
- `index.md` concepts/sources 카탈로그 갱신.
- 결정 근거: 위키의 공백이 "심판이 *누가*(독립성)"는 3개 페이지에 있으나 "*어떻게* 채점하나(신호 해상도)"가 부재 → 그 자리를 채우는 concept 승격(사용자 승인). 프로젝트 독립성 원칙으로 호스트 적용(재라벨 병합 정책 등)은 위키 제외, 일반 방법론만 수록. preprint 미검증이라 프레이밍·수치는 "저자 보고"로 표기, 로짓 전제 없이 성립하는 K·C 는 재사용 가치로 강조. `sources/labeling/` 배치(kulikov autodata 옆, 검증·데이터 품질 계열).

## [2026-07-08] lint | 수록 원칙 누출 전면 정리 (Sonnet 서브에이전트 4개 병렬 점검)
- 전 38개 페이지 6축 점검. 구조축(끊어진 링크·고아·미인용 concept·역링크 비대칭·index 누락) 전부 0건, 낡은 주장 0건.
- **수록 원칙(프로젝트 독립성) 위반 5개 파일 수정**: 호스트 실제 함수명 `extract_spans_from_bio` 4곳(`bio-tagging.md` 2·`sources/ner/kmou`·`sources/ner/park`) → "표준 BIO→span 변환기"로 일반화. "(본 프로젝트 관점)" 절 제목 3곳(`kmou`·`park`·`pan`) 재작성, 호스트 구현 세부(`joiner=""`·KLUE `PS`/`LC`/`OG` 매핑 프레이밍) 언어중립 이론으로 환원. `agent-skills.md` 호스트 `CLAUDE.md` 원칙명·"베트남어 라벨러" 구체 참조 일반화.
- **경미 3건**: `context-engineering.md` bare 파일명 링크 7곳 → `concepts/` 접두어. 60%(품질 저하 시작)↔70%(정리 트리거) 갭 미설명 → `token-management.md`에 인과 한 줄 보강. `inter-annotator-agreement.md` substantial 하한 0.60→0.61(Landis-Koch 스케일 정합).
- **미수정(설계상 정당)**: 소스 템플릿 필드명 이탈(`**매체**`·주제별 절 제목·GitHub 카탈로그 저자/연도 부재)은 논문 템플릿을 데이터셋·카탈로그에 글자강제하면 정보 손실이라 보류. 개념 번호-TOC 스타일도 schema 비강제.

## [2026-07-08] ingest | 김상국 2026 컨텍스트 엔지니어링 (우아한형제들)
- `concepts/context-engineering.md` 신규 (LLM 읽기 3기제[관련도 어텐션·lost-in-the-middle·유한 윈도우]·저술 Do/Don't 11항·가이드라인 vs 하네스·모델 세대별 조정·아첨/확률성 대응).
- `sources/dev/kim-2026-context-engineering.md` 신규 (원문 요약).
- 교차 참조: source ↔ 신규 concept. 신규 concept → `dynamic-workflows`(하네스의 오케스트레이션 자매 층)·`token-management`(전술 구체화)·`session-handoff`(lost-in-the-middle=context rot)·`bio-tagging`(few-shot=형식 고정 인스턴스) 발신. 역방향 backlink: `dynamic-workflows` §관련 개념·`token-management` §관련 개념(신설)·`session-handoff` §개요·`bio-tagging` §1.2에 추가.
- `index.md` concepts/sources 카탈로그 갱신.
- 결정 근거: token-management(운영 전술)·dynamic-workflows(오케스트레이션)와 구분되는 *인지 원리+저술* 레이어라 신규 concept 승격(현 12→13, 임계 15 미만). 하네스 두 용법(프롬프트 층 vs 오케스트레이션 층)을 층위로 구분해 상보 배치. dev/ topic(개발·에이전트 계열).

## [2026-06-12] lint | agent-skills ADR 권고 제거
- `concepts/agent-skills.md` §4 documentation-and-adrs 삭제: ADR(`docs/decisions/NNNN-*.md`) *생성*을 지시하는 운영 가드레일이라 프로젝트 독립 위키가 담을 내용이 아니고, 현 프로젝트의 ADR 폐지와도 어긋났음(삭제된 `docs/decisions/`를 재생성시킬 떠다니는 지시문). 보조 스킬 번호 5~9 → 4~8 재정렬.
- `sources/dev/addyosmani-agent-skills.md` 카탈로그 목록의 `documentation-and-adrs`는 상류 저장소 실재 항목이라 유지(사실 보고).
- 그 외 축 직전 점검과 동일: 끊어진 링크 0·고아 0·source 미인용 0·역링크 불일치 0. 잔여: `schema.md` 예시 블록의 pre-reorg 경로 1건(무해, 미수정).

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

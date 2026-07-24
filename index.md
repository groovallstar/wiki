# Index

전체 페이지 카탈로그. 임베딩 없이도 LLM 이 관련 페이지를 빠르게 찾도록 돕는다.

## Concepts

- `concepts/agentic-data-generation.md` — 에이전트 기반 합성 데이터 생성(데이터 과학자 루프·난이도 격차 게이트·메타 최적화)
- `concepts/bio-tagging.md` — BIO/IOB 태그 체계와 토큰 단위별 Span 변환
- `concepts/data-splitting.md` — train/dev/test·train-dev·OOD·temporal 분할과 평가 설계
- `concepts/inter-annotator-agreement.md` — 라벨러 합의도(κ 계열)와 NER/LLM 맥락 응용
- `concepts/kb-anchor-verification.md` — 외부 지식 베이스를 앵커로 삼은 라벨 독립 검증
- `concepts/agent-skills.md` — Claude Code 의 Agent Skills 운영 패턴
- `concepts/token-management.md` — LLM 컨텍스트 토큰 운영 전략
- `concepts/session-handoff.md` — 세션 전환 시 맥락 유실을 막는 4계층 핸드오프 전략
- `concepts/src-layout-packaging.md` — Python src-layout 과 editable install
- `concepts/ubiquitous-language.md` — 사용자·개발자 어휘 통일로 번역 비용을 제거하는 DDD 원칙
- `concepts/dynamic-workflows.md` — 작업별 하네스·서브에이전트 오케스트레이션·세 실패 모드와 패턴 카탈로그
- `concepts/loop-verification-gate.md` — 결정적 선통과+격리 반박자를 루프 완료 관문으로 두는 검증 방법론
- `concepts/context-engineering.md` — LLM 읽기 3기제(관련도 어텐션·lost-in-the-middle·유한 윈도우)와 컨텍스트 조성 저술 원리(Do/Don't·가이드라인 vs 하네스)
- `concepts/verifier-score-resolution.md` — LLM 심판 판정을 연속 점수로 다뤄 값·통계·기준 세 축으로 해상도를 높이는 검증 방법론

## Sources

소스는 주제별 하위 폴더로 구분한다.

### `ner/` — NER 태깅 스킴·데이터셋

- `sources/ner/ramshaw-marcus-1995-bio-tagging.md` — BIO/IOB 태그 스킴의 원조 논문
- `sources/ner/park-2021-klue.md` — 한국어 KLUE 벤치마크(음절 단위 NER 사례)
- `sources/ner/pan-2017-wikiann.md` — WikiANN 다국어 silver NER 구축
- `sources/ner/kmou-ner-dataset.md` — 한국해양대 NER 데이터셋(형태소 단위 + 비표준 BIO)
- `sources/ner/ng-2018-ml-yearning.md` — train/dev/test·train-dev 4분할·optimizing/satisficing 지표
- `sources/ner/quinonero-candela-2009-dataset-shift.md` — dataset shift 정식화(covariate/prior/concept)
- `sources/ner/recht-2019-imagenet-generalize.md` — 새 테스트셋 일반화·adaptive overfitting 실증
- `sources/ner/dwork-2015-reusable-holdout.md` — adaptive data analysis·holdout 재사용 타당성
- `sources/ner/bergmeir-benitez-2012-timeseries-cv.md` — 시계열 CV vs walk-forward(OOS) 평가
- `sources/ner/koh-2021-wilds.md` — 실세계 분포 시프트 OOD 벤치마크

### `labeling/` — 라벨링 품질 평가·검증

- `sources/labeling/cohen-1960-kappa.md` — Cohen's κ 공식의 원조 논문
- `sources/labeling/vrandecic-2014-wikidata.md` — Wikidata 데이터 모델·P31 원조 논문
- `sources/labeling/kulikov-2026-autodata.md` — Agentic Self-Instruct 자동 데이터 과학자(난이도 격차 게이트)
- `sources/labeling/kwok-2026-llm-as-verifier.md` — LLM-as-a-Verifier: 로짓 기댓값 연속 점수·세 스케일링 축(preprint)

### `env/` — 개발 환경·패키징

- `sources/env/python-editable-install-research.md` — Python editable install 동작 조사

### `dev/` — 개발 도구·에이전트

- `sources/dev/addyosmani-agent-skills.md` — Addy Osmani 의 Agent Skills 논의
- `sources/dev/evans-2003-ddd.md` — DDD 원서, ubiquitous language 정의의 1차 출처
- `sources/dev/pocock-2026-skills.md` — Matt Pocock 의 Skills For Real Engineers 카탈로그
- `sources/dev/jeong-2026-vibe-coding-token-management.md` — vibe coding 맥락의 토큰 관리
- `sources/dev/epril-2026-session-context-handoff.md` — 세션 간 컨텍스트 핸드오프 4계층 전략
- `sources/dev/anthropic-2026-dynamic-workflows.md` — Claude Code 동적 워크플로·하네스 패턴 카탈로그(Anthropic 블로그)
- `sources/dev/kim-2026-context-engineering.md` — 컨텍스트 엔지니어링 원리(어텐션·lost-in-the-middle)와 Do/Don't (우아한형제들 기술블로그)

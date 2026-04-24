# Index

전체 페이지 카탈로그. 임베딩 없이도 LLM 이 관련 페이지를 빠르게 찾도록 돕는다.

## Concepts

- `concepts/bio-tagging.md` — BIO/IOB 태그 체계와 토큰 단위별 Span 변환
- `concepts/inter-annotator-agreement.md` — 라벨러 합의도(κ 계열)와 NER/LLM 맥락 응용
- `concepts/kb-anchor-verification.md` — 외부 지식 베이스를 앵커로 삼은 라벨 독립 검증
- `concepts/agent-skills.md` — Claude Code 의 Agent Skills 운영 패턴
- `concepts/token-management.md` — LLM 컨텍스트 토큰 운영 전략
- `concepts/session-handoff.md` — 세션 전환 시 맥락 유실을 막는 4계층 핸드오프 전략
- `concepts/src-layout-packaging.md` — Python src-layout 과 editable install

## Sources

소스는 주제별 하위 폴더로 구분한다.

### `ner/` — NER 태깅 스킴·데이터셋

- `sources/ner/ramshaw-marcus-1995-bio-tagging.md` — BIO/IOB 태그 스킴의 원조 논문
- `sources/ner/park-2021-klue.md` — 한국어 KLUE 벤치마크(음절 단위 NER 사례)
- `sources/ner/pan-2017-wikiann.md` — WikiANN 다국어 silver NER 구축
- `sources/ner/kmou-ner-dataset.md` — 한국해양대 NER 데이터셋(형태소 단위 + 비표준 BIO)

### `labeling/` — 라벨링 품질 평가·검증

- `sources/labeling/cohen-1960-kappa.md` — Cohen's κ 공식의 원조 논문
- `sources/labeling/vrandecic-2014-wikidata.md` — Wikidata 데이터 모델·P31 원조 논문

### `env/` — 개발 환경·패키징

- `sources/env/python-editable-install-research.md` — Python editable install 동작 조사

### `dev/` — 개발 도구·에이전트

- `sources/dev/addyosmani-agent-skills.md` — Addy Osmani 의 Agent Skills 논의
- `sources/dev/jeong-2026-vibe-coding-token-management.md` — vibe coding 맥락의 토큰 관리
- `sources/dev/epril-2026-session-context-handoff.md` — 세션 간 컨텍스트 핸드오프 4계층 전략

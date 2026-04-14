# Wiki

프로젝트 독립적인 개발·기술 지식을 LLM이 직접 유지·갱신하는 위키. **Andrej Karpathy의 LLM-Wiki** 모델을 따른다.

> 상세 스키마: [`schema.md`](./schema.md)

## 구조

```
wiki/
├── schema.md         # 구조·운영 규칙 (이 파일의 기반)
├── concepts/         # 위키 계층: 개념·방법론·전략 페이지
└── sources/          # 원문 소스 계층: 논문·기사·블로그 요약
```

## 3계층 아키텍처

| 계층 | 역할 | 위치 |
|------|------|------|
| **원문 소스** | 불변의 외부 자료(논문·기사·데이터) 요약 노트 | `sources/` |
| **위키** | LLM이 생성·관리하는 구조화 지식 | `concepts/` |
| **스키마** | 구조·컨벤션 정의 | `schema.md` |

## 수록 원칙

1. **프로젝트 독립성** — 특정 저장소의 파일 경로·스크립트·결과 파일명이 등장하지 않는다.
2. **개발·기술 내용** — 재사용 가능한 지식(이론·알고리즘·방법론·원문 요약)만 담는다.
3. **LLM 관리 대상** — `concepts/`는 인제스트·린트 과정에서 계속 갱신된다.

## 3대 작업 (Operations)

- **인제스트(Ingest)**: 새 소스를 읽고 `sources/`에 요약, 관련 `concepts/`를 연쇄적으로 갱신.
- **쿼리(Query)**: `concepts/` 기반 답변, 근거로 `sources/` 인용.
- **린트(Lint)**: 주기적으로 모순·낡은 주장·고아 페이지·끊어진 링크 점검.

각 작업의 절차는 `schema.md`를 참조.

## 사용

다른 저장소에서 위키를 참조하려면 submodule로 연결:

```bash
git submodule add <이 repo URL> docs/wiki
git submodule update --init --recursive
```

이후 위키 수정은 `docs/wiki/`에서 직접 커밋·푸시하고, 호스트 저장소는 `git add docs/wiki && git commit`으로 pin 커밋을 갱신한다.

## 운영 스킬 (`docwiki`)

이 위키는 `.claude/skills/docwiki/SKILL.md` 스킬로 운영된다. 아래 트리거 키워드가 감지되면 Claude Code가 스킬을 자동 호출해 `schema.md` 규칙대로 작업한다.

| 작업 | 트리거 키워드 예시 | 동작 요약 |
|------|--------------------|-----------|
| **Ingest** | "위키에 추가", "위키 업데이트", "위키에 넣어", "wiki에 추가", "wiki ingest" | 원문 요약 → `sources/<slug>.md` 작성 → 관련 `concepts/` 연쇄 갱신 → `index.md`·`log.md` 기록 |
| **Query** | "wiki 조회", "wiki query", "wiki에서 X 찾아줘" | `concepts/` 우선 검색 → 인용구 포함 답변 → 재사용 가치 있으면 `concepts/`에 환원(복리) |
| **Lint** | "wiki 린트", "wiki lint" | 전 페이지 스캔 → 고아/끊어진 링크/프로젝트 경로 누출 등 6개 항목 리포트·수정 |

### 설계 메모

- 스킬은 **키워드 트리거**로 호출된다. 일반 프롬프트에서는 위키를 자동 참조하지 않는다 — 필요할 때만 읽는다.
- 스킬 본체는 `.claude/skills/docwiki/SKILL.md`, 데이터 규칙은 `schema.md`, 상위 규칙은 저장소 루트의 `CLAUDE.md §Wiki 운영`. 세 곳 모두 schema 우선 원칙으로 정합성을 유지한다.
- OMC의 범용 `wiki` 스킬(`.omc/wiki/` 대상)과는 별개 시스템이다. 섞지 않는다.

## 참고

- LLM-Wiki 개념 원문: https://news.hada.io/topic?id=28208
- Karpathy gist 원문: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- 페이지 템플릿·교차 참조 규칙: [`schema.md`](./schema.md)

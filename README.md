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

## 참고

- LLM-Wiki 개념 원문: https://news.hada.io/topic?id=28208
- 페이지 템플릿·교차 참조 규칙: [`schema.md`](./schema.md)

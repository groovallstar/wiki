# Vrandečić & Krötzsch (2014) — Wikidata: A Free Collaborative Knowledgebase

- **저자**: Denny Vrandečić, Markus Krötzsch
- **연도**: 2014
- **매체**: Communications of the ACM, 57(10), 78–85
- **DOI**: https://doi.org/10.1145/2629489
- **유형**: 1차 문헌 (지식 베이스 설계 논문)

## 핵심 요지

Wikidata 프로젝트의 **데이터 모델·거버넌스·기술 아키텍처**를 정립해 발표한 논문. Wikipedia 의 자연어 본문과 달리, 모든 엔티티에 언어 독립 식별자(Q-ID)와 구조화된 속성(property-value claim)을 부여해 기계 가독 KB 를 구축한다.

## 본 논문이 확립한 핵심 요소

### 1. 언어 독립 식별자 (Q-ID)

모든 엔티티(`Hanoi`, `Albert Einstein` 등)에 전역 고유 식별자 `Q<숫자>` 를 부여. 다언어 Wikipedia 간 교차 참조가 이 식별자로 중앙집중화된다.

### 2. Property-value claim 모델

엔티티의 사실을 `<subject, property, value>` 트리플로 기록. 속성에도 `P<숫자>` 식별자가 부여된다.

### 3. 핵심 타입 속성 P31 / P279

- **P31 (instance of)** — 이 엔티티가 어느 "종류"에 속하는가
- **P279 (subclass of)** — 한 "종류"가 상위 종류의 하위 클래스인가

예: `Q1858 (Hanoi) P31 Q5119 (capital city)` / `Q5119 P279 Q515 (city)`.

### 4. 협업 편집과 출처 기록

모든 claim 에 references 속성을 붙여 증거 출처를 기록. 커뮤니티가 편집·검증.

### 5. Wikipedia 연결

각 Wikipedia 언어판 페이지가 Q-ID 에 연결되어, `vi.wikipedia.org/wiki/Hà Nội` ↔ `Q1858` ↔ `en.wikipedia.org/wiki/Hanoi` 자동 매핑이 성립.

## 본 논문이 하지 않은 것

- NER·entity linking 시스템 구현 (→ 후속 연구: NECKAr, spaCy EL, BLINK 등)
- P31 기반 타입 분류기 학습 (→ Geiß & Gertz 2017 NECKAr)
- LLM 출력 검증 방법 (→ 2020년대 distant supervision 계열)

이 논문은 **KB 의 자체 설계**를 다루며, 그것을 활용한 downstream 검증·분류 기법은 모두 후속 문헌에 속한다.

## 인용하는 위키 페이지

- `concepts/kb-anchor-verification.md` — KB 앵커 검증 방법론 전반

# Geiß et al. (2018) — NECKAr: A Named Entity Classifier for Wikidata

- **저자**: Johanna Geiß, Andreas Spitz, Michael Gertz
- **연도**: 2018 출판 (GSCL 2017 발표)
- **매체/학회**: Language Technologies for the Challenges of the Digital Age, LNCS 10713, 115–129쪽
- **링크**: https://link.springer.com/chapter/10.1007/978-3-319-73706-5_10
- **열람일**: 2026-10-28
- **유형**: 1차 문헌

## 핵심 요지

Wikidata의 세부 타입을 location·person·organization의 세 범주로 묶는 NECKAr와 그 결과 데이터셋을 제시한다. P31(instance of)과 P279(subclass of)를 이용하여 상위 타입에 속하는 엔티티를 찾는다.

## 주요 내용

- §4는 2016-12-05 Wikidata 덤프에서 8,842,103개 항목을 추출·분류했다고 보고한다. 이 값은 현재 KB 규모나 임의 코퍼스의 커버리지 추정치가 아니다.
- §5는 P31 누락, 관련 하위 클래스 누락, KB 간 분류 차이로 인한 한계를 설명한다.
- KB 타입 유도의 사례이며 LLM 라벨러와 KB 증거의 통계적 독립성이나 자동 라벨의 정확도를 보장하지 않는다.

## 인용하는 위키 페이지

- `concepts/kb-anchor-verification.md` — P31/P279 기반 타입 유도와 스냅샷별 결과의 구분

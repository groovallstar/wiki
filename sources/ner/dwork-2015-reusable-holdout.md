# Dwork 외 (2015) — The Reusable Holdout

- **저자**: Cynthia Dwork, Vitaly Feldman, Moritz Hardt, Toniann Pitassi, Omer Reingold, Aaron Roth
- **연도**: 2015
- **매체/학회**: Science 349(6248):636-638
- **링크**: https://www.science.org/doi/10.1126/science.aaa9375
- **유형**: 1차 문헌

## 핵심 요지

같은 holdout(검증셋)을 **적응적으로**(이전 결과를 보고 다음 분석을
고르며) 반복 사용하면, 분석을 별개 부분집합에서 수행하더라도 통계적
타당성이 깨지고 거짓 발견(false discovery)이 누적된다 —
**adaptive data analysis** 문제. differential privacy에 기반해 holdout을
여러 번 안전하게 재사용하는 메커니즘을 제시한다.

## 주요 내용

- **adaptive overfitting의 이론적 근거**: 검증셋에 대고 "기법 적용 →
  점수 확인 → 다음 기법 선택"을 반복하면, 그래디언트가 아니라 *분석가의
  선택*을 통해 holdout에 서서히 과적합된다. 점수가 점점 낙관적으로
  편향된다.
- **reusable holdout 기법**: holdout을 직접 노출하지 않고 집계 정보에만
  잡음을 섞어 질의하게 하여, 적응적 재사용에도 일반화 보장을 유지한다.
- **실무 함의**:
  - 모델 선택·하이퍼파라미터 탐색은 검증셋(dev)에서, 최종 판정은
    test에서 *드물게* — test를 자주 보면 그것도 dev로 전락한다.
  - 지표를 *실험 전에* 고정(pre-registration)하면 적응적 선택의
    자유도를 줄여 거짓 발견을 억제한다.
- 단, *얼마나* 과적합되는지는 상황에 따라 다르며, 실증적으로는 분포
  시프트가 더 큰 요인일 수 있다 (→ `sources/ner/recht-2019-imagenet-generalize.md`).

## 인용하는 위키 페이지

- `concepts/data-splitting.md` — test/dev 재사용과 adaptive overfitting, pre-registration 근거

# Bergmeir & Benítez (2012) — On the Use of Cross-Validation for Time Series Predictor Evaluation

- **저자**: Christoph Bergmeir, José M. Benítez
- **연도**: 2012
- **매체/학회**: Information Sciences 191:192-213
- **링크**: https://www.sciencedirect.com/science/article/abs/pii/S0020025511006773
- **유형**: 1차 문헌

## 핵심 요지

시간 의존(time-dependent) 데이터에서 평가용 데이터를 어떻게 떼느냐를
다룬다. 표준 랜덤 k-fold 교차검증은 시계열의 시간 순서를 깨고 미래
정보가 학습에 새는(leakage) 문제가 있어, 전통적으로는 **시계열 끝
구간을 떼는 out-of-sample(OOS) 평가 = walk-forward** 방식을 쓴다.
저자들은 정상(stationary) 시계열에 한해 **blocked CV**가 단일 OOS보다
데이터 효율이 높고 더 정확한 추정을 준다는 것을 보인다.

## 주요 내용

- **랜덤 CV의 위험**: 시계열을 무작위로 섞어 분할하면 미래 시점이
  train에, 과거가 test에 섞여 시간적 누수가 발생 → 과대평가.
- **OOS / walk-forward**: 과거로 학습, 미래로 평가. 배포 현실(항상 미래를
  예측)을 그대로 시뮬레이션. 단점은 평가가 끝 구간 1회로 제한돼 데이터
  활용이 비효율적.
- **blocked CV**: 시간 블록 단위로 폴드를 구성하면, 정상 시계열에서는
  순서를 크게 깨지 않으면서 여러 번 평가해 분산을 줄일 수 있다.
- **함의**: 시간에 따라 분포가 변하는 데이터(개념 표류 포함)의 평가
  분할은 *반드시 시간 순서를 존중*해야 한다. 재학습 파이프라인의
  "과거 학습 → 다음 기간 예측" 루프가 이 원칙의 실무 형태다.

## 인용하는 위키 페이지

- `concepts/data-splitting.md` — temporal / walk-forward 분할, 시간 누수 방지

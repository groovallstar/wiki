# Ng (2018) — Machine Learning Yearning

- **저자**: Andrew Ng
- **연도**: 2018
- **매체/학회**: deeplearning.ai (무료 기술 서적)
- **링크**: https://www.deeplearning.ai/machine-learning-yearning/
- **유형**: 2차 자료 (실무 방법론 서적)

## 핵심 요지

ML 프로젝트의 *데이터 분리·지표 설계*를 실무 관점에서 정리한 가이드.
두 가지 기여가 데이터 분리 주제의 핵심이다 — (1) **dev/test는
배포(타겟) 분포와 같아야 하고 train은 달라도 된다**, (2) train과
dev/test 분포가 다를 때 **train-dev set**으로 분산(variance)과
데이터 불일치(data mismatch)를 분리 진단한다.

## 주요 내용

- **3분할의 역할 분리**: train(학습) / dev=validation(튜닝·모델 선택) /
  test(최종 1회 판정). test로 튜닝하면 test가 dev로 전락한다.
- **dev·test 동일 분포 = 타겟 분포**: 사람은 dev/test를 향해 최적화하므로,
  그 둘은 *실제로 잘하고 싶은 분포*(배포 환경)를 대표해야 한다. train은
  증강·외부 데이터로 부풀려 분포가 달라도 무방하다.
- **단일 평가 지표(single-number evaluation metric)**: 순위를 매길 숫자
  하나를 먼저 정한다. 비교 마비를 막는다.
- **optimizing vs satisficing 지표**: 최적화 지표 1개 + 나머지는 만족
  임계값(가드레일). 예) 정확도 최적화, 지연시간 ≤ 임계값.
- **train-dev set (4분할 진단)**: train과 dev/test 분포가 다르면, train에서
  떼되 학습엔 안 쓰는 train-dev를 둔다. 오차 격차를 분해:
  - optimal(사람 수준) → train: **avoidable bias**
  - train → train-dev: **variance** (같은 분포, 안 본 데이터)
  - train-dev → dev: **data mismatch** (분포가 바뀌는 지점)
  - dev → test: **dev 과적합**
- **data mismatch 처방**: 수동 오류 분석으로 train과 dev/test 차이를 파악,
  train을 dev/test에 더 닮게(데이터 수집·합성) 만든다. 일반적 정칙화로는
  해결되지 않는다.

## 인용하는 위키 페이지

- `concepts/data-splitting.md` — train/dev/test, train-dev 4분할, dev/test=타겟 분포, optimizing/satisficing 지표

# Quiñonero-Candela 외 (2009) — Dataset Shift in Machine Learning

- **저자**: Joaquin Quiñonero-Candela, Masashi Sugiyama, Anton Schwaighofer, Neil D. Lawrence (eds.)
- **연도**: 2009
- **매체/학회**: MIT Press (Neural Information Processing series)
- **링크**: https://mitpress.mit.edu/9780262545877/dataset-shift-in-machine-learning/
- **유형**: 1차 문헌 (편저 단행본)

## 핵심 요지

**dataset shift(데이터셋 시프트)** = 학습 단계와 테스트 단계에서 입력·출력의
결합 분포 P(x, y)가 달라지는 현상. 표준 train/test 분할이 암묵적으로
기대는 IID 가정을 정면으로 위반하며, 랜덤 분할로 측정한 성능이 배포
환경에서 무너지는 근본 원인이다.

## 주요 내용

- **정의**: train과 (라벨 없는) 배포 데이터가 같은 분포가 아니다.
- **유형 분류** (Moreno-Torres 외 2012, *A unifying view on dataset shift
  in classification*, Pattern Recognition 45(1):521-530 에서 정식화):
  - **covariate shift**: P(x) 변화, P(y|x) 불변 (입력 분포만 이동)
  - **prior probability shift**: P(y) 변화 (클래스 비율 이동)
  - **concept shift / drift**: P(y|x) 변화 (입력→정답 관계 자체가 이동)
- **함의**: "모델이 안 좋다"가 아니라 "평가 분포가 배포 분포와 다르다"가
  성능 저하의 원인일 수 있다. 처방은 모델 정칙화가 아니라 분포 정렬
  (재가중·도메인 적응·타겟 분포 데이터 수집)이다.
- transfer learning·domain adaptation·active/semi-supervised learning을
  dataset shift 대응의 한 갈래로 묶어 조망한다.

## 인용하는 위키 페이지

- `concepts/data-splitting.md` — IID 가정·분포 시프트 3유형, OOD 평가의 이론적 배경

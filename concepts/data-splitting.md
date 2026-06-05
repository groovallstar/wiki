# 데이터 분리 (Data Splitting & Evaluation Design)

> 평가의 타당성은 데이터를 *어떻게 나누느냐*에서 결정된다. train/dev/test
> 3분할은 토대일 뿐이고, 핵심은 dev/test가 **배포 분포를 대표**하고 분할이
> **암기가 아니라 일반화**를 측정하게 만드는 것이다.

## 개요

지표(F1, accuracy 등)는 데이터 분할만큼만 타당하다. 같은 분포에서 무작위로
쪼갠 IID(independent and identically distributed, 독립 동일 분포) 분할은
*낙관적 상한*을 잰다. 실세계 데이터는 거의 항상 학습 분포와 다르기 때문이다
(`sources/ner/koh-2021-wilds.md`, `sources/ner/quinonero-candela-2009-dataset-shift.md`).
따라서 분할 설계는 "데이터를 3등분"이 아니라 "*무엇을 측정할지*를 분할로
인코딩하는" 작업이다.

## 핵심 원칙

1. **3분할의 역할 분리** — train(학습) / dev=validation(튜닝·모델 선택) /
   test(최종 1회 판정). test로 튜닝하면 test가 dev로 전락한다.
2. **dev·test = 타겟(배포) 분포, train은 달라도 된다** — 사람은 dev/test를
   향해 최적화하므로 그 둘이 *실제로 잘하고 싶은 분포*를 대표해야 한다.
   train은 증강·합성·외부 데이터로 부풀려 분포가 달라도 무방하다.
3. **train-dev로 분산 vs 데이터 불일치를 분해(4분할)** — train과 dev/test
   분포가 다르면 train에서 떼되 학습엔 안 쓰는 train-dev를 둔다.
4. **IID vs OOD** — 실세계 ≈ OOD(out-of-distribution). ID 점수가 아니라
   ID/OOD *격차*(robustness gap)가 배포 견고성의 진짜 지표다.
5. **분포 시프트 3유형** — covariate(P(x)) / prior(P(y)) / concept(P(y|x)).
   성능 저하가 모델이 아니라 *분포 차이* 때문일 수 있다.
6. **시간 의존 데이터는 temporal/walk-forward 분할** — 과거 학습, 미래 평가.
   랜덤 분할은 미래→과거 누수를 일으킨다.
7. **holdout 반복 사용은 adaptive overfitting 위험** — 단, 이론적 위험과
   실증 결과는 갈린다(아래 §세부).

## 세부 설명

### 4분할 진단 — variance인가 distribution shift인가

train과 dev/test 분포가 다를 때, 단순 train↔dev 격차를 "분산"이라 부르면
틀린다(분포가 이미 다르므로). train-dev set으로 분해한다
(`sources/ner/ng-2018-ml-yearning.md`):

| 격차 | 이름 | 의미 |
|------|------|------|
| optimal(사람 수준) → train | avoidable bias | 과소적합·용량 부족 |
| train → train-dev | **variance** | 같은 분포 안 본 데이터에서 하락 = 과적합 |
| train-dev → dev | **data mismatch** | 분포가 바뀌는 지점 = distribution shift |
| dev → test | dev 과적합 | 검증셋에 맞춰짐 |

variance가 크면 처방은 정칙화·데이터 증가, mismatch가 크면 train 분포를
타겟에 닮게 만들기 — 처방이 정반대라 분해가 선행해야 한다.

### 분포 시프트 (dataset shift)

표준 분할이 기대는 IID 가정을 깨는 현상
(`sources/ner/quinonero-candela-2009-dataset-shift.md`):
- **covariate shift**: 입력 분포 P(x)만 이동, P(y|x) 불변
- **prior probability shift**: 클래스 비율 P(y) 이동
- **concept shift/drift**: 입력→정답 관계 P(y|x) 자체가 이동(시간에 따른 표류)

실세계 배포는 ID 대비 성능이 크게 떨어지며 이는 보편적이다
(`sources/ner/koh-2021-wilds.md`). 그래서 ID test와 별도로 **도메인·시간
단위 OOD test**를 두고 격차를 본다.

### temporal / walk-forward 분할

시간 의존 데이터를 무작위로 섞으면 미래 시점이 train에 새어 과대평가된다.
시간 끝 구간을 떼는 out-of-sample(walk-forward) 평가가 배포 현실(항상
미래 예측)을 시뮬레이션한다(`sources/ner/bergmeir-benitez-2012-timeseries-cv.md`).
"과거로 학습 → 다음 기간 예측"의 재학습 루프가 이 원칙의 실무 형태다.
주의: walk-forward 지표는 매 기간 *다른 test*로 재므로, 평평한 지표가
모델 정체인지 기간 난이도 변화인지 분리해서 봐야 한다(고정-test 학습곡선 +
동결 모델 시계열 평가로 분해).

### adaptive overfitting — 이론 vs 실증

- **이론**(`sources/ner/dwork-2015-reusable-holdout.md`): 같은 holdout에
  대고 "결과 보고 다음 분석 선택"을 반복하면, 분석가의 선택을 통해 holdout에
  서서히 과적합되어 점수가 낙관적으로 편향된다. → 지표를 *실험 전에*
  고정(pre-registration), test는 드물게 본다.
- **실증**(`sources/ner/recht-2019-imagenet-generalize.md`): 새 테스트셋
  실험에서 정확도 하락의 주범은 적응적 과적합이 아니라 *분포 시프트*였고,
  모델 순위와 게인은 보존됐다. → "test 재사용 = 자동 과적합"은 입증되지
  않았으며, 분포 시프트가 더 큰 요인일 수 있다.
- 결론: 둘을 나란히 둔다 — adaptive overfitting은 *방어할 위험*이되,
  관측된 ID/OOD 격차의 1차 설명은 보통 분포 시프트다.
- **합성 데이터 루프에서의 동형**: held-out 점수에 대고 *데이터 생성 규칙*을
  반복 조정하면 생성 하네스가 holdout에 adaptive overfitting 한다. 에이전트가
  격차 지표를 속이는 보상 해킹(solver 프롬프트 조작 등)도 같은 Goodhart 계열
  실패다 — 최종 판정 세트는 사전 고정하고 드물게 본다
  (`concepts/agentic-data-generation.md`).

### 지표 선택과의 연결

분할이 *무엇을* 재는지 정하면, 그 위에서 **단일 최적화 지표 1개 +
나머지 가드레일(satisficing)** 로 의사결정을 구조화한다
(`sources/ner/ng-2018-ml-yearning.md`). 평가 천장의 기준자로는
사람 간 합의도(IAA)를 쓴다 — 모델 성능이 IAA에 닿으면 그것이 실질
상한이다(`concepts/inter-annotator-agreement.md`).

### NER 적용 메모

위 일반화 원칙을 NER에 대입하면, train/test의 **엔티티 표면형 중복**을
제거한 분할(seen vs novel 엔티티)이 암기와 일반화를 분리한다. 표면형이
겹치면 "삼성=ORG" 같은 사전 암기를 일반화로 오인한다. (이 NER 특화
분할의 전용 출처는 후속 ingest 대상.)

## 출처

- `sources/ner/ng-2018-ml-yearning.md` — train/dev/test, train-dev 4분할, dev/test=타겟 분포, optimizing/satisficing 지표
- `sources/ner/quinonero-candela-2009-dataset-shift.md` — IID 위반, 분포 시프트 3유형
- `sources/ner/koh-2021-wilds.md` — IID vs OOD, OOD 평가셋·robustness gap
- `sources/ner/bergmeir-benitez-2012-timeseries-cv.md` — temporal/walk-forward 분할, 시간 누수
- `sources/ner/dwork-2015-reusable-holdout.md` — holdout 재사용·adaptive overfitting·pre-registration
- `sources/ner/recht-2019-imagenet-generalize.md` — adaptive overfitting 실증 반례, ID 점수의 낙관성

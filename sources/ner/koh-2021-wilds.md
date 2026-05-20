# Koh 외 (2021) — WILDS: A Benchmark of in-the-Wild Distribution Shifts

- **저자**: Pang Wei Koh, Shiori Sagawa, Henrik Marklund 외
- **연도**: 2021
- **매체/학회**: ICML 2021
- **링크**: https://arxiv.org/abs/2012.07421
- **유형**: 1차 문헌

## 핵심 요지

실세계에서 자연스럽게 발생하는 분포 시프트를 담은 10개 데이터셋
벤치마크. 모든 데이터셋에서 **표준 학습은 in-distribution(ID)보다
out-of-distribution(OOD) 성능이 현저히 낮다**는 것을 일관되게 보여,
ID 점수만으로는 배포 성능을 알 수 없음을 실증한다.

## 주요 내용

- **현실적 시프트 유형**: 병원 간(종양 분류), 카메라 트랩 간(야생동물),
  시간·지역 간(위성 영상) 등 — 도메인·시간·장소 축의 분할.
- **ID/OOD 격차가 크고 보편적**: 같은 분포에서 쪼갠 test로는 잘 나오는
  모델이 다른 도메인·시점에서 큰 폭으로 떨어진다 → 랜덤 분할은 낙관적
  상한.
- **명시적 OOD 분할의 필요**: 평가 분할을 *도메인/시간 단위*로 구성해야
  배포 견고성을 측정할 수 있다. ID test와 OOD test를 함께 두고 그
  격차(robustness gap)를 지표로 본다.
- domain generalization(도메인 일반화) 연구의 표준 평가 기반.

## 인용하는 위키 페이지

- `concepts/data-splitting.md` — IID vs OOD, OOD 평가셋 구성, ID/OOD robustness gap

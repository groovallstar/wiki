# Recht 외 (2019) — Do ImageNet Classifiers Generalize to ImageNet?

- **저자**: Benjamin Recht, Rebecca Roelofs, Ludwig Schmidt, Vaishaal Shankar
- **연도**: 2019
- **매체/학회**: ICML 2019
- **링크**: https://arxiv.org/abs/1902.10811
- **유형**: 1차 문헌

## 핵심 요지

CIFAR-10·ImageNet의 *새 테스트셋*을 원 수집 절차를 그대로 복제해
만들고 기존 모델들을 재평가했다. 정확도가 일제히 하락(CIFAR-10
3~15%p, ImageNet 11~14%p)했지만, **그 하락의 원인은 테스트셋 반복
재사용에 의한 adaptive overfitting이 아니라 분포 시프트(약간 더 어려운
이미지)** 였다는 것이 핵심 결론.

## 주요 내용

- **test 재사용 = 자동 과적합, 은 입증되지 않았다**: 수년간 같은
  벤치마크를 재사용했음에도, 새 테스트셋에서 모델 *순위*는 보존됐고
  적응적 과적합의 흔적(diminishing returns)은 관찰되지 않았다.
- **게인은 전이된다(오히려 증폭)**: 원 테스트셋에서의 개선이 새
  테스트셋에서 *더 큰* 개선으로 나타났다 → 리더보드 진보는 순위
  의미에서 "진짜"다.
- **단, 절대 수치는 낙관적**: 새 분포(재현하기 어려운 미세한 분포 차이)로
  가면 절대 성능은 떨어진다 → in-distribution 점수는 상한이다.
- adaptive overfitting의 *이론적* 위험(→ `sources/ner/dwork-2015-reusable-holdout.md`)과
  *실증* 결과가 갈리는 대표 사례. 위키엔 둘을 나란히 기록한다.

## 인용하는 위키 페이지

- `concepts/data-splitting.md` — adaptive overfitting의 실증 반례, in-distribution 점수의 낙관성, 게인 전이

# Inter-Annotator Agreement (IAA)

> 목적: 두 명 이상의 라벨러(사람·모델 불문)가 동일 데이터에 붙인 어노테이션이 얼마나 합의하는지를 **우연 보정**으로 정량화하는 방법. NER/분류 데이터셋 품질 평가와 LLM silver 데이터 신뢰도 측정의 중심 도구.

## 목차

1. [왜 "우연 보정"이 필요한가](#1-왜-우연-보정이-필요한가)
2. [Cohen's kappa — 두 라벨러 · 명목 척도](#2-cohens-kappa--두-라벨러--명목-척도)
3. [해석 스케일 — Landis-Koch](#3-해석-스케일--landis-koch)
4. [kappa paradox — 왜 NER 토큰 단위에서 κ가 붕괴하는가](#4-kappa-paradox--왜-ner-토큰-단위에서-κ가-붕괴하는가)
5. [NER/span 태스크에서의 실전 운용](#5-nerspan-태스크에서의-실전-운용)
6. [관련 지표 — Fleiss, Krippendorff, Gwet](#6-관련-지표--fleiss-krippendorff-gwet)
7. [LLM 자동 라벨 품질 검증에서의 역할](#7-llm-자동-라벨-품질-검증에서의-역할)
8. [출처](#출처)

---

## 1. 왜 "우연 보정"이 필요한가

두 라벨러의 단순 일치율(percent agreement, `po`)만 보면 **우연에 의한 일치도 포함**되어 값이 부풀려진다. 예를 들어 라벨이 두 개(Yes/No)뿐이고 라벨러 A·B가 모두 90%의 데이터에 Yes를 찍으면, **서로 독립적으로 무작위 라벨링해도 `0.9×0.9 + 0.1×0.1 = 0.82`의 기대 일치**가 나온다. `po = 0.85` 같은 값은 우연보다 조금 나은 수준에 불과하다.

이 우연 기여분을 **주변 확률(marginal probability)의 내적**으로 추정한 것이 `pe`이며, 이를 뺀 뒤 "우연을 넘어선 일치 여지"로 정규화한 지표가 kappa다.

## 2. Cohen's kappa — 두 라벨러 · 명목 척도

Cohen (1960) 이 제안한 계수:

```
κ = (po − pe) / (1 − pe)

po = 실제 일치율
pe = Σ_k p_A(k) · p_B(k)   (각 범주 k에 대한 A·B 주변 확률의 내적)
```

값 해석:
- `κ = 1` → 완전 일치
- `κ = 0` → 우연 수준 (po = pe)
- `κ < 0` → 우연보다도 못한 일치 (구조적 불일치)

**전제**: 범주는 서로 배타적이고 순서 정보가 없는 명목(nominal) 척도여야 한다. 서열(ordinal) 정보를 활용하려면 weighted kappa, 연속 척도는 ICC(intraclass correlation), 복수 annotator는 Fleiss/Krippendorff로 확장된다.

## 3. 해석 스케일 — Landis-Koch

Cohen 논문은 절대 임계치를 제시하지 않는다. 현장 관행의 수치 해석은 **Landis & Koch (1977)** 가 의학 연구용 휴리스틱으로 제안한 것이다:

| κ 범위 | 해석 |
|---|---|
| < 0.00 | poor |
| 0.00–0.20 | slight |
| 0.21–0.40 | fair |
| 0.41–0.60 | moderate |
| **0.61–0.80** | **substantial** |
| 0.81–1.00 | almost perfect |

**주의**: 이 스케일은 도메인 무관 절대 기준이 아니다. 저자도 구간 구분이 자의적임을 명시했다. Gold·silver의 수용 임계치는 이 표만으로 정하지 않고 태스크와 오류 비용에 맞춰 정한다.

## 4. kappa paradox — 왜 NER 토큰 단위에서 κ가 붕괴하는가

Feinstein & Cicchetti (1990) 가 지적한 **kappa paradox**: 주변 분포가 한쪽으로 극단적으로 쏠리면 po가 매우 높아도 pe 도 덩달아 높아져 κ 가 작아진다.

NER에서 `O`(엔티티 아님) 토큰이 많은 코퍼스는 이 문제에 주의해야 한다. 두 라벨러의 `O` 비율이 높으면 `pe`도 커질 수 있다. `O` 비율이나 κ의 범위를 보편적인 수치로 가정하지 않고 실제 주변 분포와 일치율을 함께 보고한다.

토큰 단위 κ 하나로 NER 품질을 판단하지 않고, 측정하려는 불일치에 맞춰 다음 방법을 검토한다:

1. **Span 단위 pair 구성** — 두 라벨러가 붙인 span의 offset 합집합에 대해 `(A의 타입, B의 타입)` 쌍을 만들고, 한쪽만 태그한 span은 상대를 `O`로 간주할 수 있다. 이는 양쪽 모두 엔티티를 표시하지 않은 영역을 제외하는 조건부 측정이므로 토큰 κ와 직접 비교하지 않는다. 경계 일치 규칙도 명시한다.
2. **Span-level pairwise F1** — 한 라벨러를 기준으로 다른 라벨러의 span을 평가한다. exact/partial match와 집계 방식을 명시하며, F1은 우연 보정 지표가 아니라는 점을 구분한다.
3. **Krippendorff α** — 누락·부분 일치·연속 척도까지 일반화된 지표 (아래 §6).

## 5. NER/span 태스크에서의 실전 운용

### 5.1 두 가지 불일치 축 분리

같은 span에 대해 라벨러가 붙일 수 있는 불일치는 두 축으로 나뉜다:

- **Coverage mismatch** — "이것을 엔티티로 볼 것인가" (recall 차이). 한쪽만 태그한 경우가 모두 여기 속한다.
- **Type confusion** — "엔티티임은 합의하나 타입이 다른 경우" (PS vs OG 등).

둘을 분리해 보고해야 문제의 원인이 드러난다. Coverage mismatch 가 크면 **가이드라인이 모호하거나 라벨러 숙련도가 낮다**는 신호이고, Type confusion 이 크면 **스키마 자체가 모호하다**는 신호다.

### 5.2 confusion matrix + per-type agreement

κ 값 하나로는 정보가 부족하다. 다음을 함께 집계하는 것이 표준:
- 전체 κ 및 po·pe
- 타입별 per-type agreement ratio (`agreed / total`)
- 타입 간 off-diagonal confusion matrix (어느 타입이 어느 타입으로 혼동되는지)
- "한쪽만 O" 케이스의 비율 (coverage mismatch 크기)

### 5.3 수용 기준

Gold·silver 전체에 공통으로 적용할 κ 임계치는 이 페이지의 출처로 정당화되지 않는다. 측정 단위·라벨 분포·오류 비용과 수작업 오류 분석을 바탕으로 수용 기준을 정한다. 합의도는 모델 성능의 엄격한 상한도 아니다(`sources/labeling/richie-2022-iaa-not-ceiling.md`).

## 6. 관련 지표 — Fleiss, Krippendorff, Gwet

| 지표 | 라벨러 수 | 척도 | 특징 |
|---|---|---|---|
| Cohen κ | 2명 고정 | 명목 | 가장 간단, 사실상 NLP 표준 |
| Fleiss κ | N명 | 명목 | 모든 항목에 같은 수의 라벨러 필요 |
| Krippendorff α | 가변 N | 명목·서열·간격·비율 모두 | 누락 허용, 복잡 태스크 선호 |
| Gwet AC1/AC2 | 2명+ | 명목·서열 | prevalence paradox 완화 설계 |

지표마다 우연 일치의 모형과 거리 함수가 다르므로, 여러 지표의 수치를 같은 척도로 해석하지 않는다.

## 7. LLM 자동 라벨 품질 검증에서의 역할

LLM 으로 대량 silver 학습 데이터를 생성할 때 κ 가 쓰이는 전형적 구조:

```
동일 코퍼스
  ├─ LLM A 재라벨 → 결과 A
  └─ LLM B 재라벨 → 결과 B
       │
       └─ span offset 합집합으로 pair 구성 → κ 계산
            │
            ├─ 사전 정의한 품질 기준 충족 → 합의 부분을 silver 후보로 검토
            └─ 불일치 span → 리뷰 후보로 분리
```

한계와 보완:
- **편향 공유 위험** — A·B 가 같은 pretrain 코퍼스를 공유하면 κ 가 부당하게 높게 나올 수 있다. `concepts/kb-anchor-verification.md` 같은 **이질적 증거**를 병행해야 한다.
- **모델 간 관대함 차이** — 한쪽이 recall 이 넓고 다른 쪽이 보수적이면 coverage mismatch 가 커진다. κ 만 보면 "합의 안 됨"으로 보이지만, type confusion 이 극소하면 스키마는 건강하다는 의미다.
- **Landis-Koch 적용의 함정** — 의학 휴리스틱이라 NLP 태스크의 품질 요구 수준에 맞추려면 독자 기준을 세워야 한다.
- **연속 점수 심판으로의 이동** — 점수 토큰 확률의 기댓값 등으로 점수를 연속화해도 간격·비율 척도가 되거나 동점이 없어지는 것은 아니다(`concepts/verifier-score-resolution.md`). 점수 차이의 의미를 정한 뒤 Krippendorff α의 거리 함수나 ICC의 전제를 검토하여 합의도 지표를 선택한다.

## 출처

- `sources/labeling/cohen-1960-kappa.md` — κ 공식과 명목 척도 정의의 원조
- `sources/labeling/landis-koch-1977-agreement.md` — 해석 구간과 자의적 구분이라는 주의사항
- `sources/labeling/feinstein-cicchetti-1990-kappa-paradoxes.md` — 주변 분포 불균형과 kappa paradox
- `sources/labeling/richie-2022-iaa-not-ceiling.md` — IAA는 모델 성능의 엄격한 상한이 아니라는 모의실험 반례

### 인용 확인 보류

기존의 “Klie et al. 2024, Counting on Consensus” 귀속은 확인되지 않아 이 페이지의 근거로 사용하지 않는다. 동명 논문은 Joseph James (2026), https://arxiv.org/abs/2603.06865 로 확인되지만, 별도 원문 수록 전에는 기존 주장의 근거로 대체하지 않는다.

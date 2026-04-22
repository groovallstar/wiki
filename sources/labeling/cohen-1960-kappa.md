# Cohen (1960) — A Coefficient of Agreement for Nominal Scales

- **저자**: Jacob Cohen
- **연도**: 1960
- **매체**: Educational and Psychological Measurement, 20(1), 37–46
- **DOI**: https://doi.org/10.1177/001316446002000104
- **유형**: 1차 문헌 (통계 지표 원조)

## 핵심 요지

두 관찰자(annotator)가 **명목(nominal) 척도**로 분류한 결과의 일치도를 측정하는 계수 κ(kappa)를 제안한 논문. 단순 일치율(`po`)은 우연 일치까지 포함해 부풀려진다는 문제를 지적하고, 우연 일치(`pe`)를 보정한 지표를 정의했다.

## 본 논문의 정의

각 관찰자가 K 개의 배타적 범주 중 하나를 고른 N 건의 관찰에 대해:

- `po` = 두 관찰자의 실제 일치 비율
- `pe` = 각 범주의 주변 확률 곱의 합 (두 관찰자가 독립적으로 무작위 분류 시 기대되는 일치율)
- **`κ = (po − pe) / (1 − pe)`**

해석:
- `κ = 1` — 완전 일치
- `κ = 0` — 우연 수준 (po = pe)
- `κ < 0` — 우연보다도 못한 일치

**전제**: 범주는 서로 배타적이고 순서 정보가 없는 명목 척도. 논문은 절대 임계치를 제시하지 않는다 (Landis-Koch 해석 스케일은 후속 문헌의 기여).

## 본 논문이 하지 않은 것

- 척도 임계치 제시 (→ Landis & Koch 1977)
- 극단 분포 paradox 규명 (→ Feinstein & Cicchetti 1990)
- 복수 관찰자 일반화 (→ Fleiss 1971)
- 누락·부분 일치 일반화 (→ Krippendorff α)

이 논문의 기여는 어디까지나 "2인 · 명목 · 우연 보정" 단일 계수의 수학적 정의다.

## 인용하는 위키 페이지

- `concepts/inter-annotator-agreement.md` — IAA 개념 전반과 NER/LLM 맥락 응용

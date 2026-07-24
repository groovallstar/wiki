# Kwok et al. (2026) — LLM-as-a-Verifier: A General-Purpose Verification Framework

- **저자**: Jacky Kwok, Shulu Li, Pranav Atreya, Yuejiang Liu, Yixing Jiang, Chelsea Finn, Marco Pavone, Ion Stoica, Azalia Mirhoseini (Stanford University · UC Berkeley · NVIDIA Research)
- **연도**: 2026 (7월, v2 07-07)
- **매체/학회**: arXiv preprint (cs.AI/CL/LG/MA/RO)
- **링크**: https://arxiv.org/abs/2607.05391
- **유형**: 1차 문헌 (preprint — 제출 직후, 재현·후속 없음. 세부 주장은 "저자 보고"로 취급)

## 핵심 요지

LM judge 를 심판으로 쓰는 표준 방식은 모델에게 **이산 점수 하나**(예: 1–5 중 하나)를 뱉게 한다. 저자는 이것이 판정 신호를 뭉개(*"collapse scoring distributions into coarse discrete scores, leading to ties and poor discrimination"*) 변별에 실패한다고 진단한다. 대안으로 **점수 토큰의 로짓 확률분포에 대한 기댓값**을 취해 연속 점수를 만들고, 이 확률적 정식화가 세 방향(점수 세분성·반복 평가·기준 분해)으로 스케일링됨을 보인다. 추가 학습 없이(training-free) 코딩·로보틱스·의료 도메인에서 SOTA 를 보고한다.

## 주요 내용

### 연속 점수 = 로짓 기댓값 (Eq. 3.1)

```
R(x, τ) = (1 / CK) · Σ_c Σ_k Σ_g  p_θ(v_g | x, c, τ) · φ(v_g)
```

- `p_θ(v_g|·)`: 모델이 점수 토큰 `v_g` 에 부여한 확률(로짓 softmax)
- `φ(v_g)`: 각 점수 토큰을 스칼라 값으로 매핑
- `C`(기준 수)·`K`(반복 수)·`G`(점수 토큰 수)로 평균

**구현 디테일**: 점수 척도는 1–20 이되 **숫자가 아니라 letter 기반**으로 둔다 — *"we use a letter-based scale instead of digits to enable logprob extraction for granularity scaling."* 숫자 토큰화의 불규칙성을 피해 상위 logprob 을 깨끗이 뽑기 위함. 실험에서 `G∈{1,4,16,20}`, `K∈{1,16}`, `C∈{1,3}`.

### 세 스케일링 축

원문: *"verification can be scaled along three dimensions: (1) score granularity, (2) repeated evaluation, and (3) criteria decomposition."*

| 축 | 늘리는 것 | 효과(저자 보고) |
|---|---|---|
| **Score Granularity (G)** | 추출 점수 토큰 개수 (1→20) | 값 분해능 상승, tie 제거 |
| **Repeated Evaluation (K)** | 독립 검증 패스 수 (1→16) | Monte-Carlo 추정, 분산 O(1/K) 감소 |
| **Criteria Decomposition (C)** | 평가 기준 개수 (코드: Specification·Output·Errors) | 신호 분해 후 앙상블 |

세 축은 직교하며 Eq. 3.1 의 세 합 기호에 각각 대응한다.

### 이산 판사의 tie 문제 (핵심 근거)

- Figure 7 캡션: *"The judge produces ties in 26.7% of comparisons at k=1 due to coarse discrete scoring."*
- Table 2 (query-optimize 사례): 이산 1–5 판사는 100회 중 **88회가 동점**. *같은* 1–5 척도에 기댓값을 취하는 것만으로 tie 가 사라짐 — 척도를 바꾸지 않고 이산→연속만으로 변별이 생긴다는 논거.
- 이산 판사 계보는 Zheng et al. (LLM-as-a-judge / MT-Bench) 를 인용.

### 헤드라인 결과 (Table 3)

| 벤치마크 | Pass@1 | Oracle Pass@N | LLM-as-a-Verifier |
|---|---|---|---|
| Terminal-Bench V2 | 83.1% | 92.1% (N=5) | **86.5%** |
| SWE-Bench Verified | 76.1% | 84.4% (N=3) | **78.2%** |
| RoboRewardBench | — | — | **87.4%** (선호도 정확도) |
| MedAgentBench | 70.2% | 75.0% (N=5) | **73.3%** |

Pass@1(단일 시도)과 Oracle Pass@N(N 후보 중 정답이 하나라도 있을 때의 상한) 사이를 검증자가 얼마나 메우는지가 이득의 척도. SWE-Bench 는 이질 정책(서로 다른 모델)이 낸 후보를 대상으로도 작동.

### 한계 (Appendix A, 원문 표현)

1. **로짓 접근 전제**: *"it assumes access to scoring-token logits, which excludes several frontier models available only through restricted APIs."* 폐쇄형 API 대응은 2단계 우회(추론 생성↔로짓 추출 모델 분리, B.6)로만 가능.
2. **기준의 수동 설계**: *"criteria decomposition could be learned or dynamically generated per domain rather than hand-designed."* 현재 C 는 손으로 지정.
3. **단일턴 제한**: *"our experiments are limited to single-turn settings"* — 장기 궤적의 다중 단계 신용 할당(per-step reward)은 미완.

## 인용하는 위키 페이지

- `concepts/verifier-score-resolution.md` — 이 소스가 정의하는 핵심 방법론 페이지 (연속 점수·세 해상도 축)

### 관련 개념 (이 소스를 직접 인용하진 않음)

- `concepts/loop-verification-gate.md` — 검증을 *어디서/누가* 하나(독립성)와 직교하는, *얼마나 세밀하게* 판정하나
- `concepts/inter-annotator-agreement.md` — 판정이 연속 척도가 되면 명목 κ 가 아니라 Krippendorff α/ICC 로 이동
- `concepts/agentic-data-generation.md` — verifier/judge 가 풀이를 채점하는 지점의 신호 해상도

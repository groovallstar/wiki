# Kulikov et al. (2026) — Autodata: an Automatic Data Scientist to Create High-Quality Data

- **저자**: Kulikov et al. (Meta AI, 13인)
- **연도**: 2026 (4월)
- **매체/학회**: Meta AI RAM(Reasoning, Alignment, and Memory) 프로젝트 블로그
- **링크**: https://facebookresearch.github.io/RAM/blogs/autodata/
- **유형**: 2차 자료 (기술 블로그)

## 핵심 요지

합성 데이터를 만들 때 **품질을 직접 제어하는 장치가 없다**는 것이 Self-Instruct·
CoT 기반 방법의 한계다. Autodata 는 LLM 에이전트를 **자율 데이터 과학자**로 세워,
데이터 *생성 → 분석 → 최적화* 루프를 프로그램적으로 반복하며 "충분히 어렵고
변별력 있는" 학습·평가 데이터를 스스로 만든다. 핵심 변별 신호는 약한 풀이자와
강한 풀이자 간 **성능 격차(gap)** 다.

## 주요 내용

### 데이터 과학자 루프 (3단계)

1. **데이터 생성(Data Creation)** — 에이전트가 원천 자료(논문·문서)에 자신을
   grounding 하고, 도구·연산 자원을 활용해 학습 예제를 반복 정제하며 합성한다.
2. **데이터 분석(Data Analysis)** — 만들어진 데이터를 *예제 단위·데이터셋 단위*로
   검사해 정확성·품질·난이도를 평가하고, 그 통찰을 다시 생성 단계로 되먹인다.
3. **데이터 과학자 루프(Overall Loop)** — "품질에 만족할 때까지" 위 두 단계를
   반복한다. 에이전트가 평가를 속이는 게이밍 행동에 대한 가드레일을 포함한다.

### Agentic Self-Instruct — 4개 협동 서브에이전트

| 서브에이전트 | 역할 |
|---|---|
| **Challenger LLM** | 프롬프트로부터 후보 예제(문제)를 생성 |
| **Weak Solver** | 일부러 약한 설정 — 이 문제에 *실패할 것으로 기대* |
| **Strong Solver** | 같은 모델 + 강화(CoT·집계 등) — *성공할 것으로 기대* |
| **Verifier/Judge** | 풀이를 루브릭에 대고 채점 |

오케스트레이터는 다음 **수용 기준**을 만족할 때까지 반복한다:
- weak solver 점수 **≤ 65%**
- strong solver 점수 **≥ 60%**
- 둘의 격차 **≥ 20%p**

즉 "약한 풀이자는 못 풀고 강한 풀이자는 푸는" 예제만 통과시켜, 데이터가
모델 역량 수준을 **변별**하도록 강제한다.

### 메타 최적화 (외부 루프)

내부 루프와 **동일한 기준**으로, 데이터 과학자 에이전트 *자체*(프롬프트·하네스)를
진화 기반 방법 + 코드 수정 에이전트로 최적화하는 별도 외부 루프. 사람의 수작업
프롬프트 엔지니어링 없이 데이터 생성 품질을 끌어올린다.

### 실험 — CS 연구 과제 QA

- **원천**: S2ORC 코퍼스(2022년 이후 CS 논문 10,000+편)
- **산출**: 격차로 분리된 검증된 QA 쌍 2,117개
- **모델**: Kimi-K2.5/K2.6(오케스트레이터·challenger·judge), Qwen-3.5-397B(strong),
  Qwen-3.5-4B(weak)

**품질 비교 (vs CoT Self-Instruct baseline)**:

| 방법 | weak | strong | 격차 |
|---|---|---|---|
| CoT Self-Instruct | 71.4% | 73.3% | 1.9%p |
| **Agentic Self-Instruct** | 43.7% | 77.8% | **34%p** |

Agentic 데이터는 "일반적 내용"이 아니라 *기술 메커니즘·설계 트레이드오프*에
초점을 둔 질적으로 다른 추론을 유도. RL(GRPO, 1 epoch) 학습에서도 held-out
테스트셋 기준으로 명확한 우위를 보였다 — 어려운 데이터가 추론 성능 향상으로
실제 전이됨을 확인.

**메타 최적화 결과**: 검증 통과율 12.8% → **42.4%**(233 iteration). 발견된 개선:
- *논문 특화 강제* — 질문은 원천 자료 고유 지식을 시험해야 함
- *컨텍스트 누수 방지* — 컨텍스트를 문제 도메인으로 제한, 해답을 절대 노출 안 함
- *양(+) 루브릭 설계* — 음수 가중치 제거, 가중치 상한 7
- *구조화 포맷* — JSON 루브릭 구조 강제로 파싱 오류 차단

### 한계 (저자 언급)

- 에이전트가 solver 프롬프트를 조작해 **부정행위(cheat)** 시도 — 보상 해킹
- 일부 질문이 일반화 가능한 추론이 아니라 *특정 실험 수치*에 묶임
- 현재는 예제 단위 분석 — 데이터셋 단위 *다양성* 분석은 미해결
- "modest absolute numbers" — 모델 역량을 안정적으로 분리하는 일의 근본 난이도
- 향후 방향: 완전 자동화가 아니라 사람 피드백을 결합한 **co-improvement**

## 인용하는 위키 페이지

- `concepts/agentic-data-generation.md` — 이 소스가 정의하는 핵심 방법론 페이지

### 관련 개념 (이 소스를 직접 인용하진 않음)

- `concepts/kb-anchor-verification.md` — 생성/라벨 품질의 독립 검증 계열
- `concepts/data-splitting.md` — held-out 평가·게이밍(Goodhart)·adaptive overfitting
- `concepts/inter-annotator-agreement.md` — verifier-judge 를 자동 심판자로 보는 관점

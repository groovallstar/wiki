# Anthropic (2026) — A harness for every task: dynamic workflows in Claude Code

- **저자**: Anthropic
- **연도**: 2026
- **매체/학회**: Anthropic Blog (claude.com/blog)
- **링크**: https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code
- **유형**: 2차 자료

## 핵심 요지

에이전트가 단일 고정 하네스(harness — 에이전트를 구동·제어하는 오케스트레이션 골격)에 의존하는 대신, **작업마다 자기 하네스를 즉석에서 직접 짠다**("write its own harness on the fly, custom-built for the task at hand"). 핵심은 격리된 컨텍스트 창을 가진 서브에이전트(subagent) 여럿을 오케스트레이션해, 긴 작업에서 생기는 세 실패 모드를 구조적으로 억제하는 것이다.

## 주요 내용

### 세 가지 실패 모드 (왜 하네스가 필요한가)

- **agentic laziness** (에이전트 게으름): 최소만 하고 "완료"를 선언.
- **self-preferential bias** (자기 출력 편애): 자기가 낸 답을 후하게 채점.
- **goal drift** (목표 표류): 이터레이션이 누적되며 원래 목표에서 이탈.

### 패턴 카탈로그

- **classify-and-act**: 작업 유형을 분류해 분기.
- **fan-out-and-synthesize**: 작업을 작은 단계로 쪼개 각각 에이전트를 돌리고 결과를 종합.
- **adversarial verification**: 별도로 띄운 에이전트가 산출물을 rubric/기준에 대해 *적대적으로 반증*. 자기편애를 막는 핵심 장치.
- **generate-and-filter**: 여러 후보를 만들고 품질 rubric으로 거른다.
- **tournament**: 에이전트들을 경쟁시키고 pairwise(쌍대 비교)로 심판.
- **loop until done**: 정지 조건(새 발견 없음 / 더 이상 오류 없음)에 도달할 때까지 에이전트를 반복 spawn.

### 검증 원리

- "comparative judgment is more reliable than absolute scoring" — 절대 점수보다 비교 판단이 더 신뢰성 높다.
- 오탐을 줄이려 규칙 검토에 **skeptic persona**(회의자 페르소나) 서브에이전트를 활용.
- 근본원인 조사: 서로 disjoint(겹치지 않는)한 증거에서 가설을 세운 에이전트들이 **검증자·반박자 패널**(panel of verifiers and refuters)을 거친다.

### 사례

- **Bun**의 Zig→Rust 재작성: 서브에이전트가 개별 요소를 고치고 병합 전 적대적 리뷰.
- **deep research**: 웹 검색 fan-out → 소스 fetch → 주장 적대적 검증 → 인용 포함 리포트 합성.
- **근본원인 조사**: 가설 생성 후 검증자·반박자 패널.

### 코인된 용어

harness, subagent, worktree(격리 실행 환경), agentic laziness, self-preferential bias, goal drift.

## 인용하는 위키 페이지

- `concepts/dynamic-workflows.md`
- `concepts/loop-verification-gate.md`

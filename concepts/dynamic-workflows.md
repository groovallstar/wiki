# Dynamic Workflows (작업별 하네스)

> 에이전트가 작업마다 격리 컨텍스트 서브에이전트를 오케스트레이션하는 하네스를 즉석에서 구성해, 긴 작업의 세 실패 모드(게으름·자기편애·목표 표류)를 억제한다.

## 개요

단일 고정 하네스(harness — 에이전트를 구동·제어하는 오케스트레이션 골격) 하나로 모든 작업을 처리하면, 작업이 길고 복잡해질수록 품질이 무너진다. 동적 워크플로는 **작업 유형에 맞춰 하네스 자체를 그때그때 구성**하고, 격리된 컨텍스트 창을 가진 서브에이전트 여럿에게 일을 분산·검증시킨다.

## 핵심 원칙

긴 작업에서 단일 에이전트가 빠지는 세 실패 모드를 구조로 막는 것이 목적이다.

- **agentic laziness** (게으름): 최소만 하고 완료 선언 → 명시적 완료 기준 + 타자(他者) 검증으로 대응.
- **self-preferential bias** (자기 출력 편애): 자기 답을 후하게 봄 → **격리 컨텍스트의 적대적 검증**으로 대응(핵심).
- **goal drift** (목표 표류): 누적되며 목표 이탈 → 원래 기준을 동결하고 매 이터레이션 대조.

## 세부 설명

### 패턴 카탈로그

- **classify-and-act**: 작업 유형 분류 → 분기.
- **fan-out-and-synthesize**: 분할 → 병렬 실행 → 종합.
- **adversarial verification**: 별도 에이전트가 rubric에 대해 *반증*. 자기편애 차단.
- **generate-and-filter**: 다수 후보 생성 → 품질 거름.
- **tournament**: 경쟁 + pairwise 심판.
- **loop until done**: 정지 조건("새 발견 없음")까지 반복 spawn.

### 검증 설계 원리

- **비교 > 절대**: "이게 좋은가?"(절대)보다 "A가 B와 같은가/나은가?"(비교)가 신뢰성 높다.
- **skeptic persona**: 규칙 위반 검출 시 회의자 페르소나로 오탐을 줄인다.
- **검증자·반박자 패널**: 단일 검증자 대신 여러 독립 검증자/반박자의 합의로 판정한다.

### 격리의 의미

독립 검증을 설계할 때는 검증자가 본체의 대화 맥락을 상속하는지 확인한다. 별도 컨텍스트 창이라는 사실만으로 깨끗한 맥락을 보장하지는 않는다. 예를 들어 Claude Code의 일반 서브에이전트는 새 맥락에서 시작하지만 fork는 부모 대화를 상속한다. 격리는 자기 출력 편애를 줄이기 위한 설계 조건이며, 검증의 공정성이나 정확성을 보장하지는 않는다. 근거: `sources/dev/anthropic-claude-code-subagents.md`.

## 출처

- `sources/dev/anthropic-2026-dynamic-workflows.md` — 작업별 하네스·패턴 카탈로그·세 실패 모드 원문.
- `sources/dev/anthropic-claude-code-subagents.md` — 일반 서브에이전트와 fork의 맥락 차이

## 관련 개념

- `concepts/loop-verification-gate.md` — adversarial verification 패턴을 루프 완료 게이트로 구체화한 방법론.
- `concepts/agentic-data-generation.md` — verifier/judge 독립성(자기 심판 회피)이 self-preferential bias 대응과 동형.
- `concepts/context-engineering.md` — 하네스의 *프롬프트 저술* 층(성공 기준·검증·중단 조건). 여기의 오케스트레이션 층과 상보.
- `concepts/agent-skills.md` — 여기서 정리한 실패 모드를 겨냥하는 시판 스킬 카탈로그의 채택 판단.

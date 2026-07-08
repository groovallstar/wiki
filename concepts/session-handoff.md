# 세션 핸드오프 — 4계층 전략

> 세션 전환 시 맥락 유실을 막는 4계층 핸드오프 전략 — in-session 관리 · Document & Clear · 영구 메모리 · cross-session orchestration.

## 개요

LLM 코딩 에이전트(Claude Code 등)는 세션이 바뀌면 "어제의 결정·실패한 가설·미묘한 제약"이 증발한다. 컨텍스트 윈도우가 커져도(200K → 1M) 구조적으로는 해결되지 않는다. 핸드오프는 **세션을 일회성 대화가 아닌 지식 자산으로 전환**하려는 체계적 대응이다.

세 가지 문제를 동시에 다뤄야 한다:

- **Context rot** — 긴 컨텍스트에서 중간 정보가 덜 참조되는 "lost in the middle"(기제는 `concepts/context-engineering.md`). 경험칙상 60% 사용률 근처에서 품질 저하.
- **Auto-compact 비대칭성** — 요약이 *가장 덜 똑똑한 순간*에 발동해 다음 턴에 필요한 맥락을 잘라낸다.
- **Context amnesia** — 새 세션은 백지. 규약 문서는 "규칙"은 잇지만 "어제 왜 그 접근을 폐기했는지"는 잇지 못한다.

## 핵심 원칙

1. **한 가지 방법으로 전부 해결되지 않는다** — 시나리오별 도구가 다르다.
2. **새 task는 새 세션이 기본** — "이 세션이 많이 안다"는 유혹은 대부분 오염된 컨텍스트 비용을 과소평가한다.
3. **Handoff는 fact가 아니라 hypothesis** — 이전 세션이 잘못 요약했을 수 있다. 새 세션은 문서를 읽은 뒤 **반드시 실제 코드와 대조 검증**한다.
4. **상태 서술형 > 명령형** — `"Implement X next"`는 새 세션이 맹목적으로 실행하게 만든다. `"X is not yet implemented; depends on Y"`가 옳다.
5. **Bookkeeping은 LLM이 부담** — 사람은 방향성·검수만.

## 세부 설명

### 시나리오 → 전략 매핑

| 시나리오 | 특징 | 기본 전략 |
|---|---|---|
| In-session 연속 | 같은 task 계속 | 계속 이어가기 |
| 실패 후 재시도 | 방금 시도가 틀림 | `rewind` + 학습 반영 |
| 같은 task, 새 세션 | 컨텍스트 오염/포화 | Document & Clear |
| 관련 task, 새 세션 | 구현 → 문서화 등 | clear + brief prompt |
| 무관한 task | 완전한 전환 | clear |
| 며칠 뒤 재개 | 시간 간격 큼 | Handoff 문서 → 새 세션 |
| Multi-agent 조율 | 여러 에이전트 공존 | Registry + report 구조 |

### Tier 1 — In-Session 관리

- **Rewind (대화 되감기)**: 실패한 시도 직전으로 돌아가 "왜 안 됐는지"를 다음 프롬프트에 녹인다. 실패가 컨텍스트에서 사라지고 학습만 남는다.
- **Focus 지시가 있는 요약**: `/compact`류 요약은 focus 지시 없이 쓰지 않는다. 결과가 예측 불가능. 많은 시니어 사용자가 아예 회피하고 Tier 2로 간다.
- **Subagent 격리**: "main context를 더럽히고 싶지 않은 탐색"(웹 검색, 긴 로그 분석, 넓은 코드베이스 탐색)에 한정. 과다 생성은 조율 오버헤드로 역효과.

### Tier 2 — Short-Term Handoff (Document & Clear)

3단계:

1. 현재 상태를 `.md`로 dump하게 지시한다.
2. 세션을 초기화한다.
3. 새 세션에서 그 `.md`를 읽고 이어서 작업한다.

**Handoff 문서 필수 섹션**:

- **Summary** — 완료된 것 1–3문장
- **Key Decisions** — 결정과 근거
- **Traps to Avoid** — 실패한 접근, 빠지기 쉬운 함정 (가치가 가장 큰 섹션)
- **Working Agreements** — 사용자 선호 (예: "커밋 전에 리뷰")
- **Relevant Files** — `<path>:L10-L45 — 왜 중요한지` (라인 번호 필수)
- **Open Work** — **상태 서술형만**. 명령형 금지
- **Prompt for New Chat** — 새 세션이 붙여넣을 텍스트. 마지막에 **"나열된 파일을 실제로 Read하고 이 문서의 주장을 코드와 대조해 검증한 뒤 지시를 기다려라"**를 반드시 포함

**작성 실전 규칙**:

- 규약 문서(CLAUDE.md 등)와 중복 금지 — "Read <규약> first. Do NOT restate" 명시
- 2,000 토큰 이내. 상세는 Tier 3의 registry로 분리
- 실패를 명시적으로 기록. 성공 정보는 코드에 남지만 실패 정보는 handoff에 남기지 않으면 소실

### Tier 3 — Persistent Context (영구 메모리)

- **규약 문서 계층**: repo root / 서브디렉토리 / 사용자 홈 세 층위. **변하지 않는 것**만 담는다(아키텍처 규약, 빌드 명령, 코딩 스타일, 도메인 용어집). 변하는 상태를 섞으면 매 세션 토큰이 낭비된다. 정기 prune이 중요.
- **Report Registry 패턴**: main 에이전트는 ~50줄짜리 `_registry.md`만 읽고, 개별 리포트는 on-demand로 로드. 예시 디렉토리 구성:
  ```
  <reports-root>/
  ├── _registry.md          # 카탈로그
  ├── analysis/             # 조사·리서치
  ├── arch/                 # 결정·ADR·spec
  ├── bugs/                 # 근본 원인
  ├── commits/              # 커밋 문서
  ├── handoff/              # 세션 간 전달
  ├── impl/  review/  tests/
  └── archive/              # 완료/대체
  ```
  Multi-agent 조율 시 "agent A가 만든 리포트를 agent B가 무시하는" amnesia를 구조적으로 차단한다.
- **Memory tool (API 레벨)**: tool result가 clear되기 전에 에이전트가 스스로 요약을 메모리 파일에 남기게 한다. 자체 에이전트 구축 시 핵심 도구.

### Tier 4 — Cross-Session Orchestration

- **Spec-Driven Development**: PRD/스펙 문서 자체가 handoff 매체. 자유 대화보다 명시 문서 기반이 더 일관적이며, 인간 리뷰와 버전 관리가 함께 작동.
- **ADR (Architecture Decision Record)**: Handoff의 "Key Decisions"가 반복 주제라면 ADR로 승격. 세션 수명을 넘어 규약 문서·코드 리뷰의 기준이 된다.
- **커밋 메시지의 이중 기록**: 커밋 메시지는 "무엇·어디"를 잘 전달하지만 "왜·어떤 대안을 배제했는지"는 담지 못한다. ADR/handoff 문서와 이중 기록한다.
- **Master-Clone vs Lead-Specialist**: 특화 subagent를 많이 만들면 main이 컨텍스트를 빼앗겨 자기 코드조차 subagent를 통해야 알게 된다. 대안은 규약 문서에 맥락을 두고 main이 자기 복제본(clone)에 동적으로 위임하는 **Master-Clone** 구조. handoff 관점에서도 "특화 상태"를 따로 전달할 필요가 없어 단순하다.

### 안티패턴

- **Auto-compact 맹신** — 가장 안 좋은 순간에 발동한다.
- **파일 전체 dump** — 20줄 필요한데 2,000줄 파일 통째로 넣기.
- **Long-running 집착** — 대개 새 task는 이전 맥락의 10%만 필요하고 90%는 노이즈.
- **Subagent 과다 생성** — 조율 오버헤드가 작업을 초과.
- **Handoff 없는 무중단 세션** — 세션 초기화 사고 시 복구 불가. 매 milestone에 handoff 문서 갱신하는 습관이 일종의 백업.

## 출처

- `sources/dev/epril-2026-session-context-handoff.md` — 4계층 전략 원문 요약 (Anthropic 공식 문서·엔지니어링 블로그·커뮤니티 사례 종합)

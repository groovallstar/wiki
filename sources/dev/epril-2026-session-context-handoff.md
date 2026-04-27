# 정도현 (2026) — Claude 세션 간 Context Handoff: 맥락을 잃지 않는 4계층 전략

- **저자**: 정도현 (Toby, epril.com)
- **연도**: 2026 (2026-04-23)
- **매체/학회**: 블로그 글 (Toby's Codex)
- **링크**: https://codex.epril.com/claude-session-context-handoff-4-layer-strategy
- **유형**: 2차 자료 (Anthropic 공식 문서·엔지니어링 블로그·커뮤니티 사례 종합)

## 핵심 요지

세션 전환 시 잃어버리는 맥락(어제 폐기한 접근, 실패한 가설, 미묘한 제약)을 체계적으로 보존하기 위한 **4계층(Tier) 핸드오프 전략**. 컨텍스트 윈도우가 200K → 1M으로 커져도 사라지지 않는 세 가지 구조적 문제(Context rot / Auto-compact 비대칭성 / Context amnesia)를 동시에 해결한다.

- Tier 1: In-Session Context Management — `/rewind`, `/compact [focus]`, subagent
- Tier 2: Short-Term Handoff — **Document & Clear 패턴** (dump → `/clear` → 새 세션에서 로드)
- Tier 3: Persistent Context — CLAUDE.md 계층, Report Registry, Memory tool
- Tier 4: Cross-Session Orchestration — Spec-Driven Dev, ADR, git commit, Master-Clone 구조

## 주요 내용

### 문제의 세 층위

- **Context rot**: 긴 컨텍스트에서 중간 정보가 덜 참조되는 "lost in the middle". 실무 관찰상 ~60% 사용률에서 품질 저하 체감(공식 권고 아닌 경험칙).
- **Auto-compact 비대칭성**: 컨텍스트가 한도에 가까워진 *가장 덜 똑똑한 순간*에 요약이 발동해, 다음 턴에 필요한 맥락이 잘려 나간다.
- **Context amnesia**: 새 세션은 백지. CLAUDE.md가 규약은 잇지만 "어제 어떤 접근을 왜 폐기했는지"는 잇지 못한다.

### 시나리오 매핑 (하나의 방법으로 다 안 된다)

| 시나리오 | 기본 전략 |
|---|---|
| In-session 연속 | 계속 이어가기 |
| 실패 후 재시도 | `/rewind` + 학습 반영 |
| 같은 task, 새 세션 | Document & Clear |
| 관련 task, 새 세션 | `/clear` + brief prompt |
| 무관한 task | `/clear` |
| 며칠 뒤 재개 | Handoff 문서 → 새 세션 |
| Multi-agent 조율 | Registry + report 구조 |

Anthropic 공식 가이드는 "새 task는 새 세션" 휴리스틱을 기본으로 제시한다.

### Tier 1 — In-Session

- **`/rewind` (Esc Esc 두 번)**: 실패한 시도를 컨텍스트에서 제거하고 학습을 다음 프롬프트에 녹이는 가장 저평가된 도구.
- **`/compact [focus]`**: focus 없이 쓰면 결과가 예측 불가능. 늘 지시를 붙이는 습관이 낫다. 시니어 사용자들은 아예 회피하고 Document & Clear로 간다.
- **Subagent**: "main context를 더럽히고 싶지 않은 탐색"에만 사용. 과다 생성은 Tier 4의 함정.

### Tier 2 — Document & Clear

1. 현재 상태를 `.md`로 dump하도록 지시
2. `/clear`로 세션 초기화
3. 새 세션에서 "이 `.md`를 읽고 이어서 작업해"로 시작

**Handoff 문서 필수 섹션** (오픈소스 `/transfer-context` 스킬 기준):

- **Summary** — 완료된 것 (1–3문장)
- **Key Decisions** — 결정과 근거
- **Traps to Avoid** — 실패한 접근, 빠지기 쉬운 함정
- **Working Agreements** — 사용자 선호
- **Relevant Files** — `path:L10-L45 — 왜 중요한지`
- **Open Work** — **상태 서술형**만 허용. `"X is not yet implemented"` OK, `"Implement X next"` 금지
- **Prompt for New Chat** — 마지막에 반드시 **"나열된 파일을 실제로 Read하고 이 문서의 주장을 코드와 대조해 검증해"** 형태의 검증 지시 포함

> Handoff는 fact가 아니라 hypothesis로 다뤄야 한다. 이전 세션이 혼동 상태에서 문서를 썼다면 새 세션이 오류를 그대로 이어받는다.

보고된 효과: 10K+ 토큰의 지식 전달이 1K–2K 토큰으로 압축, 주 3–5회 전환 시 주간 30–50K 토큰 절감 (Black Dog Labs 측정).

### Tier 3 — Persistent Context

- **CLAUDE.md 계층**: repo root, 서브디렉토리, 사용자 홈. **변하지 않는 것만** 담는다(아키텍처 규약, 빌드 명령, 스타일, 용어집). 변하는 것은 매 세션 토큰 낭비.
- **Report Registry 패턴** (Ilyas Ibrahim): main agent는 **50줄짜리 registry**만 읽고, 리포트는 on-demand로 로드. 예시 디렉토리:
  ```
  <reports-root>/
  ├── _registry.md          # ~50줄 index
  ├── analysis/  arch/  bugs/  commits/  design/
  ├── handoff/   impl/   review/  tests/
  └── archive/
  ```
  Multi-agent 조율 시 agent A가 만든 리포트를 agent B가 무시하는 amnesia를 구조적으로 차단.
- **Memory tool (API 레벨)**: `clear_tool_uses_20250919` + memory tool 조합으로 tool result clear 전에 요약을 남긴다. 자체 에이전트 구축 시 핵심.

### Tier 4 — Cross-Session Orchestration

- **Spec-Driven Development**: PRD/spec 자체가 handoff 매체. 자유 대화보다 명시적 문서 기반이 훨씬 일관적.
- **ADR**: Handoff의 "Key Decisions"가 반복되면 ADR로 승격. 세션 수명을 넘어 CLAUDE.md·코드 리뷰의 기준이 된다.
- **Git commit message**: `feat: ... (ADR-0012)` 같은 메시지는 `git log --oneline`만으로도 맥락 복원에 기여. 단, "왜"는 못 담으므로 ADR/handoff 문서와 **이중 기록**.
- **Master-Clone vs Lead-Specialist** (Shrivu Shankar): Custom subagent를 많이 만들면 main이 컨텍스트를 빼앗긴다. CLAUDE.md에 맥락을 두고 main이 Task/Explore로 자기 복제본에 위임하는 Master-Clone이 더 단순하고 handoff에도 유리(특화 에이전트 상태를 따로 전달할 필요 없음).

### 작성 실전 규칙

- 명령형 금지, **상태 서술형**만
- 파일 참조는 **라인 번호까지**
- CLAUDE.md 중복 금지 — "Read CLAUDE.md first. Do NOT restate" 명시
- 실패를 명시적으로 기록 ("Traps to Avoid"가 가치를 가장 많이 올린다)
- **2,000 토큰 이내** 유지. 상세는 registry 참조

### 안티패턴

- Auto-compact 맹신
- 20줄 필요한데 2,000줄 파일 통째로 dump
- Long-running session 집착 ("이 세션이 많이 안다")
- Subagent 과다 생성 → 조율 오버헤드가 작업을 초과
- Handoff 없는 무중단 세션 → `/clear` 실수 시 복구 불가

## 인용하는 위키 페이지

- `concepts/session-handoff.md` — 본 글의 4계층 전략을 프로젝트 독립 방법론으로 정제

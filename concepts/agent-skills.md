# Agent Skills — 평가 및 적용 현황

> AI 코딩 에이전트용 스킬 카탈로그를 데이터·NLP 파이프라인 관점에서 평가하고, 강력 추천·보조·제외로 가른다.

카탈로그 원본과 링크는 §출처의 소스 페이지에 있다. 상류에서 스킬이 계속 늘어나므로 아래 평가는 **그 소스에 적힌 관측 시점 기준**이고, 시점이 갱신되면 이 페이지도 같이 봐야 한다.

---

## 강력 추천 (Strong Fit)

### 1. test-driven-development
- **요지**: 실패하는 테스트를 먼저 쓰고 최소 코드로 통과시킨 뒤 리팩터링(RED → GREEN → REFACTOR). 버그 수정 시에는 재현 테스트부터(Prove-It Pattern).
- **적용 근거**:
  - "원자적 기능 단위 / 검증 및 문서화" 원칙과 정확히 부합.
  - pytest 기반 테스트 구조가 이미 존재하는 프로젝트라면 즉시 활용 가능.
- **적용 포인트**: 신규 함수 추가 시 기본 워크플로로 채택.

### 2. debugging-and-error-recovery
- **요지**: "Stop-the-line" 규칙 — 예기치 못한 실패 발생 시 기능 추가 중단, 증거 보존, Triage 체크리스트(재현 → 격리 → 원인 → 수정 → 방어)로 근본 원인 해결.
- **적용 근거**:
  - 다중 백엔드 통합(LLM API, 모델 로딩 등)은 실패 모드가 다양 → 체계적 디버깅 프로토콜 필요.
  - 태그 정렬/span 추출 등 미묘한 버그가 평가 지표를 오염시킬 위험이 있는 NLP 파이프라인에 특히 유효.
- **적용 포인트**: 벤치마크 결과 이상치, span 평가 불일치, 라벨러 예외 발생 시 표준 절차.

### 3. performance-optimization
- **요지**: 추측 금지, 측정 먼저(MEASURE → IDENTIFY → FIX → VERIFY → GUARD). 프로파일이 지목한 병목만 손댄다.
- **적용 근거**:
  - LLM 벤치마크는 처리량·지연이 핵심이며 배치 크기, 프롬프트 길이, 샘플 수 등 튜닝 축이 많음.
  - NLP 파이프라인에서 병목은 측정 없이 예측하기 어렵다.
- **유의**: 이 스킬은 Core Web Vitals(웹 프론트엔드) 예시가 포함되어 있으나 방법론(측정 우선)은 그대로 적용 가능. 웹 지표 부분은 무시.

### 4. doubt-driven-development
- **요지**: 확신에 찬 출력이 그대로 서기 전에, 앞선 대화 맥락을 안 가진 검토자를 새로 띄워 **반증을 목표로** 교차 심문한다(CLAIM → EXTRACT → DOUBT → RECONCILE → STOP). 완성물에 대한 사후 판정이 아니라, 되돌리는 비용이 아직 쌀 때 거는 진행 중 자세다.
- **적용 근거**:
  - 긴 세션은 검증 안 된 가정을 조용히 "사실"로 굳히는데, 같은 맥락 안에서 자기 출력을 검토하면 그 굳은 가정을 그대로 물려받는다. 맥락을 끊는 것이 이 스킬의 작동 기제다.
  - 위키가 이미 같은 원리를 방법론으로 들고 있다 — `concepts/loop-verification-gate.md` 의 격리 반박자, `concepts/dynamic-workflows.md` 의 자기편애(self-preferential bias) 실패 모드. 이 스킬은 그 원리를 개인 작업 단위 절차로 내린 형태다.
- **유의**: 서브에이전트 안에서는 중첩 생성이 막혀 "신선 맥락"이 성립하지 않는다. 저자도 그 경우를 열화 모드로 표시하라고 못 박는다 — 메인 세션에서만 온전히 돈다.

---

## 보조 (Nice to Have)

### 5. spec-driven-development
- **요지**: 코드 전에 사양 작성. SPECIFY → PLAN → TASKS → IMPLEMENT 4단계 게이트, 단계마다 사람 검토.
- **적합성**: "사람 주도 분해(Human-Led Decomposition)" 원칙과 맞음. 새 언어 추가 같은 중간 규모 기능에 유용.
- **주의**: 단일 파일 수정/오타 수정에는 과도.

### 6. planning-and-task-breakdown
- **요지**: 스펙을 검증 가능한 작은 태스크로 분해. 의존성 그래프, 인수 기준, 병렬화 가능성까지 명시.
- **적합성**: "원자적 기능 단위" 원칙의 실행 도구로 사용 가능. 데이터셋 추가 + 라벨러 + 평가 파이프라인 같은 다단계 작업에 적합.

### 7. code-review-and-quality
- **요지**: 5축 리뷰 — 정확성, 가독성, 아키텍처, 보안, 성능. 승인 기준은 "완벽"이 아니라 "코드 상태를 분명히 개선하는가".
- **적합성**: PR/머지 전 체크리스트로 사용. OMC의 `code-reviewer` 에이전트 역할과 중복되므로 기준 문서로만 참조해도 충분.

### 8. code-simplification
- **요지**: 동작을 정확히 보존하면서 가독성을 높이는 리팩터링. "줄 수 감소"가 아닌 "이해 속도 향상"이 목표.
- **적합성**: 점진적으로 복잡해진 토큰 정렬·평가 러너 모듈을 정리할 때 유용.

### 9. incremental-implementation
- **요지**: 얇은 수직 슬라이스로 구현 → 테스트 → 검증 → 커밋 반복. 한 번에 ~100줄 이상 작성 전 멈추기.
- **적합성**: "Atomic Functionality" 원칙의 실행 패턴. 신규 처리 단계나 평가 지표를 추가할 때의 기본 리듬.

### 10. interview-me
- **요지**: 계획·사양·코드가 생기기 **전에** 한 번에 한 질문씩 던지되 질문마다 자기 추측을 붙여, 사용자의 다음 대답을 예측할 수 있을 정도(확신 ~95%)까지 의도를 좁힌다. 사람이 말한 것과 실제로 원하는 것의 간극을 가장 싼 시점에 메우는 절차다.
- **적합성**: 요구가 "무엇을·누구를 위해·왜 지금"에서 하나라도 비었을 때 쓴다. 착수 후에는 전환 비용이 실재해 사람이 어긋난 결과를 "이만하면 됐다"로 합리화하므로, 그 굳음이 오기 전에 거는 게 요점이다.
- **주의**: 사람이 실시간으로 답할 수 있어야 성립한다. 비대화형 실행(스케줄러·자동 루프)에서는 추측으로 메우지 말고 막힌 지점으로 보고하는 것이 저자 지침이다.

---

## 도입 방식 제안

1. **즉시 도입**: TDD, debugging-and-error-recovery — 기존 원칙과 100% 정렬, 런타임 비용 없음.
2. **작업 앞뒤에 상시**: interview-me(착수 전 의도 좁히기) → doubt-driven-development(비자명한 결정이 서기 전 반증). 둘은 같은 축의 앞뒤라 짝으로 쓸 때 값이 크다 — 앞은 *무엇을 만들 것인가*의 어긋남을, 뒤는 *만든 것이 맞는가*의 어긋남을 잡는다.
3. **다음 중간 규모 작업에 시범 적용**: spec-driven-development + planning-and-task-breakdown + incremental-implementation 세트.
4. **참조용 체크리스트로 보관**: code-review-and-quality, code-simplification, performance-optimization — 작업 트리거 시 스킬 원문 참조.

## 제외된 스킬 (참고)

UI/프론트엔드/배포 계열은 CLI·배치 기반 데이터 파이프라인 성격과 맞지 않아 일반적으로 제외 대상:
`frontend-ui-engineering`, `browser-testing-with-devtools`, `api-and-interface-design`, `shipping-and-launch`, `ci-cd-and-automation`, `deprecation-and-migration`, `security-and-hardening`, `source-driven-development`, `observability-and-instrumentation`, `context-engineering`, `idea-refine`, `using-agent-skills`.

**같은 이름이 가리키는 대상이 다른 경우** — 여기 제외된 `context-engineering` 은 카탈로그 안의 스킬 파일 하나를 말한다. 분야로서의 컨텍스트 엔지니어링은 위키가 `concepts/context-engineering.md` 로 따로 들고 있고, 그쪽은 제외 대상이 아니라 핵심 개념이다.

**계열은 맞지만 규약이 경쟁해서 뺀 것** — `git-workflow-and-versioning` 은 트렁크 기반 브랜칭과 잦은 소단위 커밋을 기본값으로 들고 온다. 그런데 브랜칭·커밋 입도는 저장소마다 이미 정해 둔 규약이 정본이라, 스킬의 기본값을 그대로 들이면 두 규약이 경쟁해 어느 쪽이 이겼는지가 커밋 이력에서만 드러난다. 원칙(원자적 커밋·되돌릴 수 있는 이력)만 참고하고 절차는 저장소 규약을 따른다.

**한때 평가했다가 걷어낸 것** — `documentation-and-adrs` 는 ADR 문서 *생성*을 지시하는 운영 가드레일이라 프로젝트 독립 위키가 담을 내용이 아니어서 뺐다. 상류 카탈로그에는 여전히 실재한다.

필요 시 위 목록에서 개별 스킬을 재평가할 수 있다.

---

## 관련 개념

- `concepts/loop-verification-gate.md` — `doubt-driven-development` 가 개인 작업 단위로 내린 원리를, 루프 완료 게이트라는 자동화 장치로 올린 형태.
- `concepts/dynamic-workflows.md` — 자기편애·게으름·목표 표류 세 실패 모드. 위 스킬들이 각각 어느 실패 모드를 겨냥하는지의 상위 지도.

## 출처

- `sources/dev/addyosmani-agent-skills.md` — 카탈로그 원본 및 수록 스킬 목록

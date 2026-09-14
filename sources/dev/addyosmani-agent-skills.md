# addyosmani/agent-skills — AI 코딩 에이전트용 스킬 카탈로그

- **저자/관리자**: Addy Osmani (Google Chrome DevRel)
- **링크**: https://github.com/addyosmani/agent-skills
- **열람일**: 2026-09-14
- **검증 기준 커밋**: `be4e44a9fbc5e8df0beaefadbb28bd22ee61cc39`
- **고정 원문**: https://github.com/addyosmani/agent-skills/blob/be4e44a9fbc5e8df0beaefadbb28bd22ee61cc39/README.md
- **신규 항목 원문**: https://github.com/addyosmani/agent-skills/blob/be4e44a9fbc5e8df0beaefadbb28bd22ee61cc39/skills/constraint-driven-development/SKILL.md
- **라이선스**: 저장소 명시
- **유형**: 2차 자료 (오픈소스 가이드 모음)

## 카탈로그 개요

Claude Code 등 AI 코딩 에이전트가 일관된 워크플로로 동작하도록 돕는 **process-driven skill** 모음. 각 스킬은 `SKILL.md` 파일에 frontmatter(name·description·trigger)와 본문(절차·체크리스트·예시)을 담는다.

## 수록 스킬 (2026-09-14 시점, 25종)

`api-and-interface-design`, `browser-testing-with-devtools`, `ci-cd-and-automation`, `code-review-and-quality`, `code-simplification`, `constraint-driven-development`, `context-engineering`, `debugging-and-error-recovery`, `deprecation-and-migration`, `documentation-and-adrs`, `doubt-driven-development`, `frontend-ui-engineering`, `git-workflow-and-versioning`, `idea-refine`, `incremental-implementation`, `interview-me`, `observability-and-instrumentation`, `performance-optimization`, `planning-and-task-breakdown`, `security-and-hardening`, `shipping-and-launch`, `source-driven-development`, `spec-driven-development`, `test-driven-development`, `using-agent-skills`.

2026-08 관측에서는 2026-04의 21종 대비 삭제된 항목 없이 3종이 늘었다 — `doubt-driven-development`(신선 맥락 반박자를 결정 앞에 세우는 절차), `interview-me`(한 번에 한 질문씩 물어 의도를 확신 ~95%까지 끌어올리는 선행 인터뷰), `observability-and-instrumentation`(로그·메트릭·트레이스를 기능과 같이 쓰는 계측 절차).

2026-09 관측에서는 `constraint-driven-development`가 추가되어 lifecycle 24종과 meta-skill 1종, 총 25종이다. 상류 README의 절 제목은 여전히 “All 24 Skills”지만 본문은 총 25종을 설명한다. 신규 스킬은 품질 기준과 측정값을 기록하고, 검사 비용에 따라 실행 위치를 정하며, 테스트 삭제·임계값 하향 등 기준 약화를 감시한다.

카탈로그는 상류에서 계속 늘어나므로 이 목록은 관측 시점과 함께 읽어야 한다. 개별 스킬의 채택 판단은 `concepts/agent-skills.md` 가 쥔다.

## 인용하는 위키 페이지

- `concepts/agent-skills.md`

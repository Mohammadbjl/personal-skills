# Using personal-skills with Antigravity

Antigravity is a first-class target for this repository. Use `AGENTS.md` as the shared project instruction file and load skills from `skills/` on demand.

## Setup

1. Clone or open this repository in Antigravity.
2. Keep `AGENTS.md` in the project root.
3. Ask the agent to consult the meta-skill before non-trivial work:

```text
Use skills/using-agent-skills/SKILL.md to decide which workflow applies, then follow the selected skill.
```

## Recommended Operating Model

Use this lifecycle:

```text
DEFINE  → spec-driven-development
PLAN    → planning-and-task-breakdown
BUILD   → incremental-implementation + test-driven-development
VERIFY  → debugging-and-error-recovery when failures occur
REVIEW  → code-review-and-quality
SHIP    → shipping-and-launch
```

## Skill Invocation Examples

```text
Follow skills/spec-driven-development/SKILL.md and write a spec for this feature before implementation.
```

```text
Follow skills/incremental-implementation/SKILL.md and implement only the next task from tasks/todo.md.
```

```text
Follow skills/debugging-and-error-recovery/SKILL.md. Reproduce the failure before proposing a fix.
```

## Personas

Use specialist personas from `agents/` when you need focused review:

- `agents/code-reviewer.md` for general code review
- `agents/security-auditor.md` for security review
- `agents/test-engineer.md` for test strategy and coverage

Example:

```text
Use agents/test-engineer.md to identify missing tests for this change.
```

## Context Guidance

- Keep `AGENTS.md` always-on.
- Load only the skill needed for the current phase.
- Load references from `references/` only when the selected skill calls for deeper checklists.
- Preserve scope discipline: do not perform unrelated cleanup while following a skill.

## Verification

Before marking work complete, require evidence from the selected skill, such as:

- tests passing
- build/typecheck succeeding
- screenshots or browser runtime evidence for UI changes
- structured review output for review tasks


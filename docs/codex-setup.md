# Using personal-skills with Codex

Codex is a first-class target for this repository. Use `AGENTS.md` for shared instructions and explicitly reference skills or personas in prompts.

## Setup

1. Clone or vendor this repository:

```bash
git clone git@github.com:Mohammadbjl/personal-skills.git
```

2. Keep `AGENTS.md` at the workspace root when possible.
3. Reference `skills/` and `agents/` files directly in Codex prompts.

## Recommended Prompt Pattern

```text
Read AGENTS.md and skills/using-agent-skills/SKILL.md.
Determine which skill applies to this task and follow it exactly.
Do not skip verification.
```

For a specific workflow:

```text
Use skills/debugging-and-error-recovery/SKILL.md to investigate this failure.
```

```text
Use agents/code-reviewer.md as a review persona for the current diff.
```

## Codex CLI for Read-Only Review

When using Codex CLI as an external reviewer, keep it read-only and pass the review prompt through stdin:

```bash
codex exec --sandbox read-only -C <repo-path> - < /tmp/review-prompt.md
```

This pattern is useful with `doubt-driven-development`, where the main agent prepares an artifact and contract, then asks Codex for a fresh-context adversarial review.

## Recommended Skill Flow

| Work type | Skills |
|---|---|
| New feature | `spec-driven-development` → `planning-and-task-breakdown` |
| Implementation | `incremental-implementation` + `test-driven-development` |
| Bug fix | `debugging-and-error-recovery` + `test-driven-development` |
| Review | `code-review-and-quality` |
| Release | `shipping-and-launch` |

## Personas

Use personas from `agents/` as focused system prompts:

- `code-reviewer` for broad engineering review
- `security-auditor` for vulnerability and threat-model review
- `test-engineer` for test coverage and test design

## Verification

Codex sessions should finish with evidence, not assertions. Ask for:

- commands run and their outputs
- tests added or updated
- failures reproduced before fixes
- review findings with file/line references
- remaining risks or follow-up tasks


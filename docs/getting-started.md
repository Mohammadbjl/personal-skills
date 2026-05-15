# Getting Started with personal-skills

`personal-skills` works with **Gemini CLI, Cline, Antigravity, and Codex**. The repo provides reusable engineering workflows (`skills/`), specialist personas (`agents/`), Gemini CLI commands (`.gemini/commands/`), and reference checklists (`references/`).

## How Skills Work

Each skill is a Markdown file named `SKILL.md`. A skill tells an agent:

- when the workflow applies
- what steps to follow
- what rationalizations to avoid
- how to verify completion with evidence

Skills are workflows, not passive reference docs. When a skill applies, the agent should follow it step by step.

## Quick Start

### 1. Clone the repository

```bash
git clone git@github.com:Mohammadbjl/personal-skills.git
cd personal-skills
```

### 2. Pick your tool

| Tool | Setup guide |
|---|---|
| Gemini CLI | [gemini-cli-setup.md](gemini-cli-setup.md) |
| Cline | [cline-setup.md](cline-setup.md) |
| Antigravity | [antigravity-setup.md](antigravity-setup.md) |
| Codex | [codex-setup.md](codex-setup.md) |

### 3. Load the meta-skill

Start with `skills/using-agent-skills/SKILL.md`. It maps task types to the right workflow.

Example prompt:

```text
Read skills/using-agent-skills/SKILL.md, decide which skill applies to my task, then follow that skill exactly.
```

### 4. Load task-specific skills on demand

Do not load every skill at once. Load the relevant workflow for the current task:

- New feature? `spec-driven-development`
- Need a plan? `planning-and-task-breakdown`
- Implementing? `incremental-implementation` + `test-driven-development`
- Debugging? `debugging-and-error-recovery`
- Reviewing? `code-review-and-quality`
- Adding or improving skills/personas? `skill-evolution-and-quality` and, for a focused report, `skill-curator`
- Shipping? `shipping-and-launch`

## Recommended Setup

### Minimal

Keep these available in your project rules or prompt snippets:

1. `using-agent-skills` — skill discovery
2. `incremental-implementation` — small, verifiable slices
3. `test-driven-development` — proof before completion
4. `code-review-and-quality` — structured review before merge

### Full Lifecycle

```text
Starting a project:  spec-driven-development → planning-and-task-breakdown
During development:  incremental-implementation + test-driven-development
Before merge:        code-review-and-quality + security-and-hardening
Before deploy:       shipping-and-launch
```

## Using Personas

The `agents/` directory contains specialist personas:

| Persona | Purpose |
|---|---|
| `code-reviewer.md` | Five-axis code review |
| `test-engineer.md` | Test strategy and coverage analysis |
| `security-auditor.md` | Vulnerability detection and hardening review |
| `skill-curator.md` | Skill/persona comparison, quality scoring, and curation decisions |

Use a persona when you need a focused perspective. For example:

```text
Use agents/code-reviewer.md as the review persona and review the current diff.

Use agents/skill-curator.md as the curation persona and compare this proposed skill with the existing repository.
```

Gemini CLI users can use the copies in `.gemini/agents/` through commands such as `/ship`.

## Using Gemini Commands

Gemini CLI users get command shortcuts from `.gemini/commands/`:

| Command | Skill invoked |
|---|---|
| `/spec` | `spec-driven-development` |
| `/planning` | `planning-and-task-breakdown` |
| `/build` | `incremental-implementation` + `test-driven-development` |
| `/test` | `test-driven-development` |
| `/review` | `code-review-and-quality` |
| `/code-simplify` | `code-simplification` |
| `/ship` | `shipping-and-launch` + persona fan-out |

Use `/planning` instead of `/plan` because `/plan` conflicts with a Gemini CLI built-in command.

## Using References

The `references/` directory contains supplementary checklists:

| Reference | Use with |
|---|---|
| `testing-patterns.md` | `test-driven-development` |
| `performance-checklist.md` | `performance-optimization` |
| `security-checklist.md` | `security-and-hardening` |
| `accessibility-checklist.md` | `frontend-ui-engineering` |

Load a reference when a skill needs more detailed examples or checklists.

## Spec and Task Artifacts

The spec and planning workflows may create working artifacts such as `SPEC.md`, `tasks/plan.md`, and `tasks/todo.md`. Treat them as living documents:

- keep them updated while work is in progress
- use them as shared context between the human and agent
- delete or ignore them before merge if your project does not want them long term

## Tips

1. Start with `spec-driven-development` for non-trivial work.
2. Use `planning-and-task-breakdown` before implementation.
3. Use `test-driven-development` for behavior changes and bug fixes.
4. Load skills selectively; too much always-on context reduces focus.
5. Use personas for independent review perspectives.

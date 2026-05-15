# personal-skills

A portable skill pack for **Gemini CLI, Cline, Antigravity, and Codex**.

This repository contains Markdown workflows, specialist personas, Gemini CLI commands, and engineering reference checklists for AI-assisted software development.

## Supported Tools

| Tool | Status | Integration |
|---|---:|---|
| Gemini CLI | First-class | Native skills from `skills/`, commands in `.gemini/commands/`, personas in `.gemini/agents/` |
| Cline | First-class | Project rules plus explicit skill/persona references |
| Antigravity | First-class | `AGENTS.md` plus on-demand skill/persona loading |
| Codex | First-class | `AGENTS.md` plus explicit skill/persona prompts and optional Codex CLI review loops |

## Repository Layout

```text
AGENTS.md                 Shared target-tool instructions
.gemini/commands/         Gemini CLI slash commands
.gemini/agents/           Gemini CLI persona definitions
agents/                   Source persona definitions
skills/                   Reusable engineering workflows
references/               Checklists and supporting references
docs/                     Setup and contribution guides
hooks/                    Optional standalone helper scripts
```

## Quick Start

1. Clone the repository:

```bash
git clone git@github.com:Mohammadbjl/personal-skills.git
cd personal-skills
```

2. Choose your setup guide:

- [Gemini CLI](docs/gemini-cli-setup.md)
- [Cline](docs/cline-setup.md)
- [Antigravity](docs/antigravity-setup.md)
- [Codex](docs/codex-setup.md)

3. Start with the meta-skill:

```text
Use skills/using-agent-skills/SKILL.md to decide which workflow applies to my task.
```

## Core Workflow

For non-trivial feature work, follow this lifecycle:

```text
DEFINE  → spec-driven-development
PLAN    → planning-and-task-breakdown
BUILD   → incremental-implementation + test-driven-development
VERIFY  → debugging-and-error-recovery when failures occur
REVIEW  → code-review-and-quality
SHIP    → shipping-and-launch
```

For skill or persona library changes, use `skills/skill-evolution-and-quality/SKILL.md` to compare overlap, apply the quality rubric, and decide whether to create, update, merge, split, reject, or ask for clarification.

## Personas

Specialist personas live in `agents/`:

- `code-reviewer` — correctness, readability, architecture, security, performance
- `security-auditor` — vulnerability detection and threat modeling
- `skill-curator` — skill/persona comparison, quality scoring, and merge/create decisions
- `test-engineer` — test strategy and coverage analysis

Gemini CLI can use copies in `.gemini/agents/`. Other tools can reference or paste the source persona files directly.

## Contributing

See [Skill Anatomy](docs/skill-anatomy.md) and `skills/skill-evolution-and-quality/SKILL.md` before adding or changing skills or personas. New docs and automation should preserve the supported-tool scope: Gemini CLI, Cline, Antigravity, and Codex only.


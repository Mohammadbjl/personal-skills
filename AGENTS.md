# AGENTS.md

This repository is maintained for **Gemini CLI, Cline, Antigravity, and Codex**. Treat the repository as a portable collection of agent workflows, personas, commands, and references for those four target tools.

## Repository Overview

`personal-skills` is a collection of senior-engineering workflows for AI coding agents. The core product is the `skills/` directory: each skill is a Markdown workflow (`SKILL.md`) with triggers, process steps, anti-patterns, and verification criteria.

Supported targets:

| Tool | Primary integration |
|---|---|
| Gemini CLI | Native skills from `skills/`, slash commands from `.gemini/commands/`, optional personas from `.gemini/agents/` |
| Cline | Project rules plus explicit references to `skills/<name>/SKILL.md` and `agents/*.md` |
| Antigravity | `AGENTS.md` plus on-demand skill/persona loading from this repo |
| Codex | `AGENTS.md` plus explicit skill/persona prompts; Codex CLI can be used for read-only cross-model review |

Do not add setup flows, CI checks, or documentation for unsupported agent tools unless the user explicitly changes the supported-tool scope.

## Skill-Driven Execution Model

### Core Rules

- If a task matches a skill, you MUST use it.
- Skills are located in `skills/<skill-name>/SKILL.md`.
- Never implement directly if a skill applies.
- Always follow the selected skill's workflow exactly, including verification.
- If more than one skill applies, use them in the required lifecycle order.

### Intent → Skill Mapping

Map user intent to skills automatically:

- Vague idea / unclear goal → `interview-me`, then `idea-refine`
- Feature / new functionality → `spec-driven-development`, then `planning-and-task-breakdown`, `incremental-implementation`, and `test-driven-development`
- Planning / breakdown → `planning-and-task-breakdown`
- Bug / failure / unexpected behavior → `debugging-and-error-recovery`, then `test-driven-development`
- Code review → `code-review-and-quality`
- Refactoring / simplification → `code-simplification`
- API or interface design → `api-and-interface-design`
- UI work → `frontend-ui-engineering` and, when runtime verification is needed, `browser-testing-with-devtools`
- Security work → `security-and-hardening`
- Performance work → `performance-optimization`
- Documentation / ADRs → `documentation-and-adrs`
- Release readiness → `shipping-and-launch`

### Lifecycle Mapping

Follow this lifecycle for non-trivial work:

- DEFINE → `spec-driven-development`
- PLAN → `planning-and-task-breakdown`
- BUILD → `incremental-implementation` + `test-driven-development`
- VERIFY → `debugging-and-error-recovery` when failures occur
- REVIEW → `code-review-and-quality`
- SHIP → `shipping-and-launch`

Gemini CLI users can invoke matching commands from `.gemini/commands/`. Cline, Antigravity, and Codex users should invoke the same lifecycle through natural-language prompts and explicit references to the relevant `SKILL.md` files.

## Execution Model for Agents

For every request:

1. Determine if any skill applies, even if the task looks small.
2. Load/read the relevant `skills/<skill-name>/SKILL.md` file.
3. Follow the skill workflow strictly.
4. Establish deliverables, success criteria, and constraints before changing files.
5. Use tests, builds, or other evidence-based verification before declaring completion.
6. Keep scope tight; do not perform unrelated cleanup.

### Anti-Rationalization

The following thoughts are incorrect and must be ignored:

- "This is too small for a skill."
- "I can just quickly implement this."
- "I'll gather context first and decide later."
- "Verification can wait until the end."

Correct behavior:

- Always check for and use applicable skills first.
- Gather only the context needed by the selected skill.
- Verify incrementally.

## Orchestration: Skills, Personas, and Commands

This repo has three composable layers:

- **Skills** (`skills/<name>/SKILL.md`) — workflows with steps and exit criteria. The *how*.
- **Personas** (`agents/<role>.md`) — specialist review roles with a perspective and output format. The *who*.
- **Commands** (`.gemini/commands/*.toml`) — Gemini CLI entry points. The *when* for Gemini users.

Composition rule: **the user or a command is the orchestrator. Personas do not invoke other personas.** A persona may apply skills, but it must not spawn or delegate to another persona.

The only multi-persona orchestration pattern this repo endorses is **parallel fan-out with a merge step** — used by `/ship` to run `code-reviewer`, `security-auditor`, and `test-engineer` as independent perspectives and synthesize their reports. If a target tool cannot run subagents in parallel, run the personas explicitly and merge the reports in the main session.

See `agents/README.md` and `references/orchestration-patterns.md` for details.

## Creating a New Skill

### Directory Structure

```text
skills/
  {skill-name}/
    SKILL.md              # Required: skill definition
    scripts/              # Optional: executable helpers only when needed
      {script-name}.sh
```

### Naming Conventions

- **Skill directory**: `kebab-case` (for example, `web-quality`)
- **SKILL.md**: always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh` (for example, `deploy.sh`, `fetch-logs.sh`)

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing what the skill does, followed by one or more "Use when" trigger conditions.}
---

# {Skill Title}

{Brief overview of what the skill does and why it matters.}

## How It Works

{Numbered workflow}

## Verification

- [ ] {Evidence-based exit criterion}
```

### Best Practices for Context Efficiency

Skills are loaded on demand. To keep target tools focused:

- Keep `SKILL.md` concise; put long reference material in separate files.
- Write specific descriptions so agents know when to activate the skill.
- Use progressive disclosure: link directly to supporting files that should be loaded only when needed.
- Prefer scripts for repeatable checks when they save context and reduce manual error.

### Script Requirements

- Use `#!/bin/bash` shebang for shell helpers.
- Use `set -e` or `set -euo pipefail` where appropriate.
- Write status messages to stderr.
- Write machine-readable output to stdout when the script is intended for automation.
- Include cleanup traps for temp files.
- Avoid target-tool-specific environment variables unless the script is explicitly documented for that tool.

## End-User Setup Targets

Document only these setup paths:

- **Gemini CLI:** `docs/gemini-cli-setup.md`
- **Cline:** `docs/cline-setup.md`
- **Antigravity:** `docs/antigravity-setup.md`
- **Codex:** `docs/codex-setup.md`

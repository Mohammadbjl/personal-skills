# Agent Personas

Specialist personas are role prompts with one perspective and one output format. They are reusable across **Gemini CLI, Cline, Antigravity, and Codex**.

| Persona | Role | Best for |
|---|---|---|
| [code-reviewer](code-reviewer.md) | Senior Staff Engineer | Five-axis review before merge |
| [security-auditor](security-auditor.md) | Security Engineer | Vulnerability detection and threat modeling |
| [test-engineer](test-engineer.md) | QA Engineer | Test strategy, coverage analysis, Prove-It tests |

## How Personas Relate to Skills and Commands

| Layer | What it is | Example | Composition role |
|---|---|---|---|
| **Skill** | Workflow with steps and exit criteria | `code-review-and-quality` | The *how* |
| **Persona** | Role with a perspective and output format | `code-reviewer` | The *who* |
| **Command** | Gemini CLI entry point or saved prompt | `/review`, `/ship` | The *when* |

The user or a command is the orchestrator. **Personas do not call other personas.** A persona may use a skill, but it must not delegate to another persona.

## Target Tool Usage

### Gemini CLI

Gemini CLI can use persona definitions from `.gemini/agents/`. This repository keeps source persona files in `agents/` and checked-in Gemini copies in `.gemini/agents/`.

- `/review` uses the `code-review-and-quality` skill for a single review pass.
- `/ship` can fan out to `code-reviewer`, `security-auditor`, and `test-engineer`, then merge their reports.

If your Gemini CLI version does not support subagents, run each persona prompt explicitly and merge the findings in the main session.

### Cline

Reference a persona directly in the task prompt:

```text
Use agents/security-auditor.md as your persona and audit the current auth changes.
```

Run multiple personas one at a time unless your environment provides safe parallel execution.

### Antigravity

Use `AGENTS.md` for always-on rules and load a persona from `agents/` when a focused perspective is needed:

```text
Use agents/code-reviewer.md to review this diff and output the standard review template.
```

### Codex

Use persona files as explicit role prompts. For external Codex CLI review, prefer read-only execution and pass prompts through stdin.

## When to Use Each Pattern

### Direct Persona Invocation

Use when you need one perspective on one artifact:

- "Review this PR" → `code-reviewer`
- "Find security issues in `auth.ts`" → `security-auditor`
- "What tests are missing for checkout?" → `test-engineer`

### Single-Persona Command or Prompt

Use when a repeated workflow wraps one persona or one skill:

- `/review` → structured code review
- `/test` → TDD / Prove-It workflow

### Parallel Fan-Out With Merge

Use only when independent perspectives can run without shared mutable state:

```text
/ship
  ├── code-reviewer    → quality report
  ├── security-auditor → security report
  └── test-engineer    → coverage report
          ↓
       merge in main session
          ↓
       go/no-go decision + rollback plan
```

This is the only multi-persona orchestration pattern this repo endorses.

## Rules for Personas

1. A persona has one role and one output format.
2. Personas do not invoke other personas.
3. A persona may apply skills relevant to its own task.
4. Every persona file ends with a "Composition" block explaining how to invoke it.
5. If a persona sees the need for another perspective, it recommends that follow-up in its report instead of delegating.

## Adding a Persona

1. Create `agents/<role>.md` using the existing frontmatter format.
2. Define role, scope, output format, and rules.
3. Add a **Composition** block.
4. Add the persona to the table above.
5. If Gemini CLI should expose it directly, add a matching `.gemini/agents/<role>.md` copy.
6. If it changes orchestration behavior, update `references/orchestration-patterns.md`.

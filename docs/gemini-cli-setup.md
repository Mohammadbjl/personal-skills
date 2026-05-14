# Using personal-skills with Gemini CLI

Gemini CLI is a first-class target for this repository. Use native Gemini skills for on-demand workflows, `.gemini/commands/` for lifecycle shortcuts, and `.gemini/agents/` for specialist personas.

## Setup

### Option 1: Install as Gemini Skills (Recommended)

Gemini CLI can auto-discover `SKILL.md` files installed into Gemini skill locations.

Install from the repository:

```bash
gemini skills install git@github.com:Mohammadbjl/personal-skills.git --path skills
```

Or install from a local clone:

```bash
git clone git@github.com:Mohammadbjl/personal-skills.git
cd personal-skills
gemini skills install ./skills --scope workspace
```

Workspace-scoped skills are installed into `.gemini/skills/` or `.agents/skills/` depending on your Gemini CLI version and configuration. User-scoped skills are installed under your Gemini user configuration directory.

Verify installation:

```text
/skills list
```

### Option 2: Use `GEMINI.md` for Always-On Context

For rules you want loaded in every Gemini session, create a focused `GEMINI.md` instead of loading every skill.

Example:

```markdown
# Project Agent Rules

Always use skills from this repository when they apply.

Start by reading:
@skills/using-agent-skills/SKILL.md

For implementation tasks, prefer:
@skills/incremental-implementation/SKILL.md
@skills/test-driven-development/SKILL.md
```

Use `/memory show` to inspect loaded context and `/memory reload` after changes.

## Recommended Configuration

### Always-On

Keep always-on context short:

- `using-agent-skills` — decide which skill applies
- a short project rules section with commands, conventions, and boundaries

### On-Demand Skills

Install these as Gemini skills so they activate only when relevant:

- `spec-driven-development`
- `planning-and-task-breakdown`
- `incremental-implementation`
- `test-driven-development`
- `code-review-and-quality`
- `debugging-and-error-recovery`
- `shipping-and-launch`

## Slash Commands

This repo ships Gemini CLI commands under `.gemini/commands/`:

| Command | What it does |
|---|---|
| `/spec` | Write a structured spec before writing code |
| `/planning` | Break work into small, verifiable tasks |
| `/build` | Implement the next task incrementally with tests |
| `/test` | Run TDD / Prove-It workflow |
| `/review` | Five-axis code review |
| `/code-simplify` | Reduce complexity without changing behavior |
| `/ship` | Pre-launch checklist with specialist persona fan-out |

> Use `/planning` instead of `/plan`; `/plan` conflicts with a Gemini CLI command name.

## Personas in Gemini CLI

Gemini CLI persona definitions live in `.gemini/agents/`:

- `.gemini/agents/code-reviewer.md`
- `.gemini/agents/security-auditor.md`
- `.gemini/agents/test-engineer.md`

The source copies live in `agents/`. Keep both in sync when changing persona behavior.

`/ship` uses the three personas as independent reviewers and then merges their reports into a single launch decision. If your Gemini CLI version does not support subagents, run the persona prompts sequentially and merge the reports in the main session.

## MCP Integration

Some skills can use MCP servers when your Gemini CLI environment supports them. For example, `browser-testing-with-devtools` can use a Chrome DevTools MCP server for runtime browser inspection.

Configure MCP servers through your Gemini CLI configuration, then follow the relevant skill's setup section.

## Usage Tips

1. Prefer on-demand skills over always-on context.
2. Keep `GEMINI.md` focused on project conventions and boundaries.
3. Use `.gemini/commands/` for lifecycle shortcuts.
4. Use `.gemini/agents/` personas for review, security, and testing perspectives.
5. Combine skills with `references/` checklists for deeper verification.

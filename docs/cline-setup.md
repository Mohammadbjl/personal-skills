# Using personal-skills with Cline

Cline is a first-class target for this repository. Use project rules for always-on guidance, then explicitly reference skills and personas when a task needs a specific workflow.

## Setup

### 1. Clone or vendor the repository

```bash
git clone git@github.com:Mohammadbjl/personal-skills.git
```

You can keep it as a shared local repository or copy the `skills/`, `agents/`, and `references/` directories into a project.

### 2. Add project rules

Create or update your Cline project rules with a short pointer to this repository:

```markdown
# Cline Project Rules

Use the workflows in `personal-skills` when they apply.

Before acting, check `skills/using-agent-skills/SKILL.md` and load the relevant skill:
- Feature work: `spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` + `test-driven-development`
- Bugs: `debugging-and-error-recovery` + `test-driven-development`
- Reviews: `code-review-and-quality`
- Releases: `shipping-and-launch`

Do not skip verification steps from the selected skill.
```

Use whichever Cline rules location your workspace supports (for example, `.clinerules` or a Cline rules file in project settings).

### 3. Invoke skills explicitly

Cline may not auto-load every `SKILL.md`, so use explicit prompts:

```text
Read skills/test-driven-development/SKILL.md and follow it to fix this bug.
```

```text
Use skills/planning-and-task-breakdown/SKILL.md to create an implementation plan before editing code.
```

### 4. Use personas for focused review

```text
Use agents/code-reviewer.md as your persona and review the current diff.
```

```text
Use agents/security-auditor.md to audit the authentication changes.
```

## Recommended Always-On Rules

Keep always-on rules small. Recommended pointers:

- `skills/using-agent-skills/SKILL.md`
- `skills/incremental-implementation/SKILL.md` for multi-file changes
- `skills/test-driven-development/SKILL.md` for behavior changes
- `skills/code-review-and-quality/SKILL.md` before merge

Load other skills only when relevant.

## Example Workflow

```text
Task: Add password reset.

1. Use spec-driven-development to define requirements.
2. Use planning-and-task-breakdown to create tasks.
3. Implement the first task with incremental-implementation and test-driven-development.
4. Use code-reviewer and security-auditor personas before shipping.
```

## Limitations

- Do not assume native skill auto-discovery unless your Cline setup provides it.
- Do not assume parallel subagent fan-out. If you need multiple persona perspectives, run them explicitly and merge the findings.
- Keep large skill files out of always-on context unless they are needed for the current task.


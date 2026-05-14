# Orchestration Patterns

Reference catalog of agent orchestration patterns endorsed by this repo for **Gemini CLI, Cline, Antigravity, and Codex**.

The governing rule: **the user or a command is the orchestrator. Personas do not invoke other personas.** Skills are workflows that a persona or main session may apply.

## Endorsed Patterns

### 1. Direct Invocation

Single persona, single perspective, single artifact.

```text
user → code-reviewer → report → user
```

**Use when:** the work is one perspective on one artifact and can be described in one sentence.

**Examples:**

- "Review this PR" → `code-reviewer`
- "Find security issues in `auth.ts`" → `security-auditor`
- "What tests are missing for checkout?" → `test-engineer`

**Cost:** one focused pass. Use this as the baseline before considering orchestration.

---

### 2. Single-Persona Command or Saved Prompt

A command or saved prompt wraps one skill/persona with repeatable instructions.

```text
/review → code-review-and-quality skill → review report
```

**Use when:** the same single-perspective workflow happens repeatedly.

**Examples in this repo:** `/review`, `/test`, `/code-simplify` for Gemini CLI.

**Anti-signal:** if the command mostly decides which persona to call, remove the routing layer and let the user invoke the persona directly.

---

### 3. Parallel Fan-Out With Merge

Multiple personas evaluate the same input independently. The main session merges their reports.

```text
                    ┌─→ code-reviewer    ─┐
/ship → fan out  ───┼─→ security-auditor ─┤→ merge → go/no-go + rollback
                    └─→ test-engineer    ─┘
```

**Use when:**

- the sub-tasks are independent
- each persona produces a different kind of finding
- the merge step is small enough for the main session
- the target tool can run parallel subagents, or the user accepts sequential emulation

**Example in this repo:** `/ship` in Gemini CLI.

**Validation checklist:**

- [ ] Can all persona passes run without ordering dependencies?
- [ ] Does each persona produce a distinct perspective?
- [ ] Will the merge step fit in the main context?
- [ ] Is the extra cost justified by release risk?

If any answer is "no," use direct invocation or a single-persona prompt.

---

### 4. User-Driven Sequential Lifecycle

The user runs phase-specific prompts or commands in order. Each step depends on the previous output.

```text
/spec → /planning → /build → /test → /review → /ship
```

**Use when:** human checkpoints matter and the workflow has dependencies.

**Examples:** the full DEFINE → PLAN → BUILD → VERIFY → REVIEW → SHIP lifecycle.

**Why not automate it fully:** a routing agent can lose nuance between hand-offs, skip human checkpoints, and add paraphrasing cost. Keep the user or explicit commands in control.

---

### 5. Research Isolation

Use a separate context or read-only pass to inspect large amounts of material and return a digest.

```text
main session → isolated research/review pass → digest → main session continues
```

**Use when:**

- the main session needs to stay focused
- the investigation result is much smaller than the input
- the target tool supports an isolated subagent or external read-only CLI review

**Examples:** large call-site discovery, ADR summarization, cross-model doubt review.

## Target Tool Compatibility

| Tool | Best-supported orchestration |
|---|---|
| Gemini CLI | `.gemini/commands/` for lifecycle prompts; `.gemini/agents/` for personas; `/ship` may use fan-out when subagents are available |
| Cline | Direct skill/persona prompting; user-driven sequential lifecycle; run multiple personas explicitly if needed |
| Antigravity | `AGENTS.md` rules plus on-demand skill/persona loading; use available subagent features only when the environment supports them |
| Codex | `AGENTS.md` plus explicit prompts; Codex CLI can provide read-only fresh-context review through stdin |

Do not document or rely on unsupported platform-specific primitives in this repository.

## Anti-Patterns

### A. Router Persona

A persona whose job is only to decide which other persona to call.

**Why it fails:**

- adds a paraphrasing hop without domain value
- hides cost and control from the user
- duplicates skill mapping already documented in `AGENTS.md`

**Instead:** improve command prompts and intent mapping.

---

### B. Persona Calling Another Persona

A `code-reviewer` that internally invokes `security-auditor` when it sees auth code.

**Why it fails:**

- personas are designed for one perspective
- hand-off summaries lose context
- output formats and responsibilities become ambiguous

**Instead:** the first persona recommends a follow-up; the user or command runs it.

---

### C. Sequential Orchestrator That Paraphrases

An agent calls `/spec`, then `/planning`, then `/build`, and summarizes between every step without human checkpoints.

**Why it fails:**

- loses user judgment at decision points
- accumulates drift through repeated summaries
- adds cost without improving correctness

**Instead:** keep sequential lifecycle steps user-driven.

---

### D. Deep Persona Trees

A coordinator calls another coordinator that calls leaf personas.

**Why it fails:**

- each layer adds latency and information loss
- leaf personas receive weaker context
- debugging the process becomes harder than doing the work

**Instead:** keep orchestration depth at one layer: command/user → personas → main-session merge.

## Decision Flow

```text
Is the work one perspective on one artifact?
├── Yes → Direct invocation.
└── No  → Will the same composition repeat?
         ├── No  → Direct invocation, ad hoc.
         └── Yes → Are sub-tasks independent?
                  ├── No  → User-driven sequential lifecycle.
                  └── Yes → Parallel fan-out with merge if the tool supports it.
```

## When to Add a New Pattern

Add a new orchestration pattern only after:

1. It has been used successfully in real work.
2. Existing patterns do not cover it.
3. The pattern has a clear artifact in this repo.
4. Its anti-pattern shadow is documented.

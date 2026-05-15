---
name: skill-evolution-and-quality
description: Curates and improves skills, personas, commands, and references by comparing candidates with repository artifacts and scoring them against a quality rubric. Use when adding, updating, merging, reviewing, or de-duplicating skills or personas.
---

# Skill Evolution and Quality

## Overview

Use this skill to keep the repository's agent workflows coherent as new skills and personas are proposed. It defines what a high-quality skill or persona is, finds overlap with existing artifacts, decides whether to create/update/merge/reject, and verifies that the final change improves the library without duplicating behavior.

This is a Markdown-first curation workflow. Do not implement embeddings, vector databases, network services, package dependencies, or long-running automation as part of this skill.

## When to Use

- Adding a new `skills/<name>/SKILL.md` workflow.
- Adding or changing a persona in `agents/` or `.gemini/agents/`.
- Comparing two or more skills/personas to decide which one is better.
- Merging a proposed workflow into an existing skill.
- Checking whether a candidate duplicates an existing command, reference, doc, skill, or persona.
- Reviewing a repository change whose main purpose is improving agent workflows.

**When NOT to use:** Do not use this for normal application feature implementation, bug fixing, runtime performance tuning, or product UI work unless the artifact being changed is a skill, persona, command, reference, or repository workflow document. Do not use it to bypass the repository lifecycle skills; this skill governs workflow-library quality, not all engineering tasks.

## Artifact Profile

Before comparing artifacts, extract a profile for each candidate and close match:

```markdown
## Artifact Profile

- Kind: skill | persona | command | reference | doc
- Path: [repository-relative path]
- Name: [frontmatter name or file name]
- User/job-to-be-done: [what recurring problem it solves]
- Trigger phrases: [when agents should activate it]
- Exclusions: [when agents should not activate it]
- Workflow summary: [ordered process, if any]
- Output contract: [report, artifact, decision, or handoff]
- Verification checklist: [observable checks]
- Dependencies/integrations: [skills, personas, commands, references, supported tools]
- Source/mirror relationship: [source file and Gemini mirror, if applicable]
```

Validation rules:

- Existing source skills must live at `skills/<skill-name>/SKILL.md` and have frontmatter `name` matching `<skill-name>`.
- Existing Gemini skill mirrors, when checked in, must live at `.gemini/skills/<skill-name>/SKILL.md` and match the source skill unless a documented exception exists.
- Existing source personas must live at `agents/<role>.md`; Gemini persona mirrors must live at `.gemini/agents/<role>.md` and match the source persona.
- Skills require evidence-based verification. Personas require one role, one output format, explicit rules, and a Composition section.
- References and docs may be supporting material, but should not be treated as directly invokable skills unless they provide a workflow.

## The Best Skill / Persona Rubric

Score each relevant artifact with evidence. Use the rubric as the source of truth for deciding what "better" means in this repository.

### Scoring Scale

For every criterion, assign a score from 0 to 4:

| Score | Meaning |
|---:|---|
| 0 | Missing, misleading, or violates repository rules. |
| 1 | Present but vague; another agent would need to invent missing steps. |
| 2 | Usable but incomplete; important edge cases or verification gaps remain. |
| 3 | Strong; clear and actionable with only minor improvements available. |
| 4 | Excellent; specific, evidence-based, easy to apply, and well integrated. |

### Weighted Quality Model

| Criterion | Weight | Applies to | Definition |
|---|---:|---|---|
| Activation clarity and discoverability | 15 | skill, persona, command | Frontmatter or description says what the artifact does and when to use it, with clear triggers and exclusions. |
| Problem and intent fit | 10 | all | The artifact solves a real recurring agent workflow problem and has a specific user/job-to-be-done. |
| Concrete workflow and sequencing | 15 | skill, command | Steps are ordered, actionable, and hard to misinterpret; the agent can execute them without guessing. |
| Output contract and handoff quality | 10 | skill, persona, command | The artifact specifies exactly what output, report, decision, or handoff should be produced and how downstream work consumes it. |
| Evidence-based verification | 15 | all | Completion criteria require observable evidence, not vibes; verification is specific enough to run. |
| Scope discipline and boundaries | 10 | all | The artifact says when not to use it, what is out of scope, and how to avoid duplicate or unrelated work. |
| Anti-rationalization and red flags | 10 | skill, persona | It names common excuses/failure modes and gives concrete signals that the workflow is being violated. |
| Repository integration and supported-tool fit | 10 | all | It fits `AGENTS.md`, supported tools, lifecycle mapping, Gemini mirrors, and persona composition rules. |
| Context efficiency and progressive disclosure | 8 | skill, doc, reference | It keeps the main artifact focused and links supporting material only when needed. |
| Safety and untrusted-data handling | 4 | all | It treats external, user, or generated content as data, asks before destructive edits, and avoids unsafe automation. |
| Maintainability and change hygiene | 3 | all | It is easy to update, avoids duplication, and keeps source/mirror files synchronized. |

Calculate a 0-100 score by weighting each applicable criterion. If a criterion does not apply to an artifact kind, omit it from the denominator and normalize the remaining weights to 100.

### Ratings

- **excellent**: score is 85 or higher and there are no must-fix issues.
- **good**: score is 70-84 and there are no critical structural gaps.
- **needs-improvement**: score is 50-69 or one important section is missing.
- **weak**: score is below 50 or required frontmatter, output format, or verification is missing.

### Must-Fix Issues

Regardless of total score, mark the artifact as must-fix when it:

- lacks required frontmatter for a skill or persona;
- has no observable verification criteria;
- duplicates an existing artifact without adding a clear improvement;
- violates supported-tool scope or repository orchestration rules;
- instructs a persona to invoke, spawn, or delegate to another persona;
- adds automation, dependencies, or external services that are not justified by the task;
- treats untrusted user-provided or generated content as authoritative without review.

## Curation Workflow

### 1. Intake the Candidate

Classify the candidate before reading the whole repository.

```markdown
## Candidate Intake

- Proposed kind: skill | persona | command | reference | doc | unclear
- Proposed path/name: [if known]
- Source of proposal: user request | existing file | generated draft | external note
- Intended user/job-to-be-done: [one sentence]
- Primary triggers: [phrases or task types]
- Desired output: [report, artifact, decision, workflow, checklist]
- Constraints: [supported tools, no automation, no dependency changes, etc.]
- Unknowns: [questions that block a decision]
```

If kind, target audience, or expected output is unclear, ask for clarification before editing files.

### 2. Inventory Repository Resources

Search the repository for related artifacts. Use focused keywords from the candidate name, triggers, intent, workflow verbs, and output contract.

Start with broad inventory:

```bash
find skills agents docs references .gemini -type f \( -name '*.md' -o -name '*.toml' \) 2>&1 | sort | cat
```

Then search by candidate terms:

```bash
grep -RInE 'candidate-term-1|candidate-term-2|candidate-term-3' skills agents docs references .gemini \
  --include='*.md' \
  --include='*.toml' 2>&1 | cat
```

Search for lifecycle or domain neighbors when the name is new but the intent may overlap:

```bash
grep -RInE 'review|quality|persona|skill|workflow|verification|merge|curat|discover|duplicate' skills agents docs references .gemini \
  --include='*.md' \
  --include='*.toml' 2>&1 | cat
```

Read only the files that are plausible matches. Do not load every file by default.

### 3. Extract Profiles for Candidate and Matches

For the candidate and each plausible match, fill the Artifact Profile. Extract evidence from:

- frontmatter `name` and `description`;
- `When to Use` / trigger sections;
- workflow steps or command behavior;
- output templates;
- verification checklists;
- rules, red flags, and rationalizations;
- references to supported tools, mirrors, or orchestration constraints.

### 4. Classify Similarity

Classify each match using repository evidence, not only the artifact name.

| Match strength | Use when |
|---|---|
| `exact-overlap` | Same trigger, same job-to-be-done, same output contract, and substantially same workflow. |
| `strong-overlap` | Same job-to-be-done or trigger, with enough shared workflow that separate artifacts would confuse users. |
| `adjacent` | Related lifecycle phase or domain, but different trigger or output contract. Cross-reference may be enough. |
| `weak` | Shares vocabulary but not intent, workflow, or output. Do not merge based on this alone. |
| `none` | No meaningful repository relationship found. |

For every `exact-overlap`, `strong-overlap`, or `adjacent` match, record:

- overlap reasons;
- unique capabilities already present;
- unique capabilities in the candidate;
- risk of merging;
- risk of keeping artifacts separate.

### 5. Score Quality

Score the candidate and the closest existing artifacts against the weighted rubric. Each score must cite evidence such as a heading, checklist item, output template, or missing section.

Use this scorecard format:

```markdown
## Quality Scorecard: [artifact path]

| Criterion | Weight | Score 0-4 | Evidence | Must-fix? |
|---|---:|---:|---|---|
| Activation clarity and discoverability | 15 |  |  |  |
| Problem and intent fit | 10 |  |  |  |
| Concrete workflow and sequencing | 15 |  |  |  |
| Output contract and handoff quality | 10 |  |  |  |
| Evidence-based verification | 15 |  |  |  |
| Scope discipline and boundaries | 10 |  |  |  |
| Anti-rationalization and red flags | 10 |  |  |  |
| Repository integration and supported-tool fit | 10 |  |  |  |
| Context efficiency and progressive disclosure | 8 |  |  |  |
| Safety and untrusted-data handling | 4 |  |  |  |
| Maintainability and change hygiene | 3 |  |  |  |

**Total:** [0-100]
**Rating:** excellent | good | needs-improvement | weak
**Must-fix issues:** [list]
```

### 6. Decide Create, Update, Merge, Split, Reject, or Ask

Use this decision table:

| Decision | Required evidence |
|---|---|
| `UPDATE_EXISTING` | An existing artifact has exact or strong overlap, and the candidate mainly fixes gaps or improves quality. |
| `MERGE_INTO_EXISTING` | Two artifacts should become one because keeping both creates duplicate triggers or conflicting guidance. |
| `CREATE_NEW` | No exact or strong overlap exists, the candidate has a clear recurring workflow, and it fits repository scope. |
| `SPLIT_CANDIDATE` | The candidate combines unrelated jobs, lifecycle phases, roles, or output contracts that should remain separate. |
| `REJECT_OR_DEFER` | The candidate duplicates without improvement, is mostly reference material, lacks a recurring workflow, violates supported-tool scope, or requires out-of-scope automation. |
| `ASK_FOR_CLARIFICATION` | User intent, target audience, merge target, or constraints cannot be inferred from evidence. |

Decision constraints:

- Do not create a new skill when a strong existing skill can be improved instead.
- Do not merge adjacent skills only because they share terminology.
- Do not convert a persona into a router. Personas may recommend follow-up perspectives, but orchestration belongs to the user or command.
- Do not add new Gemini commands unless the user explicitly asks for a command-level workflow.
- Do not edit existing unrelated skills while curating one artifact.

### 7. Produce a Merge or Creation Plan Before Editing

Before changing files, write a concise plan:

```markdown
## Curation Plan

- Decision: UPDATE_EXISTING | MERGE_INTO_EXISTING | CREATE_NEW | SPLIT_CANDIDATE | REJECT_OR_DEFER | ASK_FOR_CLARIFICATION
- Target paths: [files to create or edit]
- Preserve: [specific sections or behavior that must remain]
- Add: [new triggers, workflow steps, rules, output contract, verification]
- Rewrite: [sections needing replacement]
- Remove: [duplicates or confusing guidance]
- Risks: [scope, mirror drift, unclear trigger, unsupported-tool reference]
- Verification commands: [repo-specific checks]
- Manual verification: [rubric and persona checks]
```

### 8. Implement the Smallest Coherent Change

Follow the curation plan exactly. Keep edits scoped to the target artifact and discoverability references.

For skills:

- create or update `skills/<skill-name>/SKILL.md` first;
- include required frontmatter, trigger/exclusion guidance, workflow, output/decision contract, rationalizations or red flags, and verification;
- add supporting files only when progressive disclosure clearly requires them;
- add or update `.gemini/skills/<skill-name>/SKILL.md` when this repository keeps a checked-in mirror.

For personas:

- create or update `agents/<role>.md` first;
- keep one role and one output format;
- include review scope, rules, output template, and Composition;
- ensure the persona recommends follow-ups rather than invoking other personas;
- add or update `.gemini/agents/<role>.md` when this repository keeps a checked-in mirror.

For docs:

- add short discoverability pointers rather than duplicating the full rubric;
- preserve the supported-tool scope: Gemini CLI, Cline, Antigravity, and Codex;
- do not introduce new setup paths for unsupported tools.

### 9. Verify the Curation Change

Run structural checks that match the changed files. Typical commands:

```bash
git diff --check 2>&1 | cat
```

```bash
find skills -mindepth 1 -maxdepth 1 -type d | while read -r dir; do
  test -f "$dir/SKILL.md"
  grep -q '^---$' "$dir/SKILL.md"
  grep -q '^name:' "$dir/SKILL.md"
  grep -q '^description:' "$dir/SKILL.md"
done 2>&1 | cat
```

```bash
! grep -RInE 'C[l]aude Code|c[l]aude\.ai|\.c[l]aude|C[L]AUDE_|O[p]enCode|W[i]ndsurf|C[u]rsor|C[o]pilot|c[l]aude plugin|@a[n]thropic-ai/c[l]aude-code' . \
  --exclude-dir=.git \
  --exclude-dir=.agent-skills 2>&1 | cat
```

For source/mirror files, run exact diffs such as:

```bash
diff -u skills/<skill-name>/SKILL.md .gemini/skills/<skill-name>/SKILL.md 2>&1 | cat
diff -u agents/<role>.md .gemini/agents/<role>.md 2>&1 | cat
```

Manual verification must confirm:

- the selected decision follows the evidence;
- the changed artifact improves or preserves its rubric score;
- verification criteria are observable;
- source and mirror files are synchronized;
- no unrelated cleanup or unsupported-tool documentation was introduced.

## Curation Report Template

Use this report when presenting findings before or after edits:

```markdown
## Skill/Persona Curation Report

**Verdict:** UPDATE_EXISTING | MERGE_INTO_EXISTING | CREATE_NEW | SPLIT_CANDIDATE | REJECT_OR_DEFER | ASK_FOR_CLARIFICATION

### Candidate
- Kind:
- Path/name:
- Intended job-to-be-done:
- Primary triggers:
- Desired output:

### Closest Matches
| Path | Kind | Match strength | Overlap reasons | Unique existing value | Unique candidate value | Decision hint |
|---|---|---|---|---|---|---|

### Quality Scorecards
- Candidate score/rating:
- Existing match score/rating:
- Must-fix issues:

### Recommended Change Plan
- Target paths:
- Preserve:
- Add:
- Rewrite:
- Remove:
- Risks:

### Verification Story
- Commands to run:
- Manual checks:
- Mirror parity checks:

### Open Questions
- [Questions that block implementation, if any]

### Deferred Enhancements
- [Ideas such as automation or commands that are intentionally out of scope now]
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The new draft sounds better, so I'll replace the old skill." | A better tone is not enough. Preserve proven triggers, workflow steps, verification, and integrations unless evidence shows they should change. |
| "Searching by file name is enough." | Overlap often appears in trigger phrases, output contracts, or lifecycle phases. Search by intent and workflow terms too. |
| "Two skills can overlap; agents will figure it out." | Duplicate triggers make skill selection harder and increase inconsistent behavior. Merge or clarify boundaries. |
| "This persona can call the other personas when it needs help." | Personas do not orchestrate other personas in this repo. They recommend follow-up reviews; the user or command orchestrates. |
| "A high score means no verification is needed." | The score is a review aid, not proof. Run structural checks and document evidence. |
| "We should automate the learning system now." | First define and validate the quality model manually. Automation without a trusted rubric amplifies bad comparisons. |

## Red Flags

- A candidate has the same triggers as an existing skill but is being added as a separate skill.
- A persona has more than one role or more than one output format.
- A skill has advice but no ordered workflow or evidence-based verification.
- Documentation repeats the full rubric instead of linking to this skill as the source of truth.
- Gemini mirrors are missing or differ from source files without explanation.
- The change introduces unsupported tool setup paths or unrelated platform references.
- The curation plan edits existing skills that are only weakly related.
- The proposal requires new dependencies, external services, or semantic-search infrastructure without an explicit separate request.

## Verification

After using this skill, confirm:

- [ ] Candidate and close matches have Artifact Profiles with evidence from repository files.
- [ ] Similarity was classified as exact-overlap, strong-overlap, adjacent, weak, or none with reasons.
- [ ] Candidate and relevant existing artifacts were scored against the weighted rubric.
- [ ] The create/update/merge/split/reject/ask decision follows the decision constraints.
- [ ] A curation plan lists target paths, preserve/add/rewrite/remove items, risks, and verification commands before edits.
- [ ] Changed skills/personas keep required frontmatter, workflow or output format, boundaries, red flags/rationalizations, and evidence-based verification.
- [ ] Source and Gemini mirror files match when mirrors are present.
- [ ] Discoverability docs point to this skill without duplicating the full rubric.
- [ ] Validation commands were run or explicitly marked not applicable, with evidence recorded.
- [ ] No unrelated cleanup, unsupported-tool setup, dependency, or automation changes were introduced.
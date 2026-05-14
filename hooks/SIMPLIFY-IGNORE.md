# simplify-ignore helper script

Block-level protection for simplification workflows. Mark code that should not be simplified; compatible wrappers can hide those blocks from the model while preserving them on disk through a backup/restore cycle.

These helpers are **standalone scripts**. They are not tied to any unsupported agent platform. To use them, your target-tool wrapper must pass compatible JSON payloads to `hooks/simplify-ignore.sh`.

## Annotate Protected Blocks

```js
/* simplify-ignore-start: perf-critical */
// manually unrolled XOR — 3x faster than a loop
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
result[2] = buf[2] ^ key[2];
result[3] = buf[3] ^ key[3];
/* simplify-ignore-end */
```

## Cache Location

Backups are stored in:

```text
.agent-skills/simplify-ignore-cache/
```

Override the project root with:

```bash
AGENT_SKILLS_PROJECT_DIR=/path/to/project
```

Add `.agent-skills/` to `.gitignore`.

## Payload Contract

The script reads JSON from stdin:

```json
{
  "tool_name": "Read",
  "tool_input": {
    "file_path": "src/example.js"
  }
}
```

Recognized `tool_name` values:

| Tool name | Action |
|---|---|
| `Read` | Back up file, replace protected blocks with placeholders in-place |
| `Edit` | Expand placeholders, save model changes, re-filter protected blocks |
| `Write` | Expand placeholders, save model changes, re-filter protected blocks |
| empty / omitted | Restore all backed-up files; use for session cleanup or crash recovery |

## How It Works

Each protected block is content-hashed and replaced with a placeholder such as:

```text
BLOCK_de115a1d: perf-critical
```

The original content is stored in `.agent-skills/simplify-ignore-cache/`. When an edit/write payload arrives, the script expands placeholders back to the original code, preserves the model's surrounding changes, and re-filters the file.

## Crash Recovery

If a session ends without cleanup and files still contain `BLOCK_<hash>` placeholders, restore manually:

```bash
echo '{}' | bash hooks/simplify-ignore.sh
```

## Annotation Syntax

```js
/* simplify-ignore-start */
/* simplify-ignore-start: reason */
/* simplify-ignore-end */
```

Any comment style works (`//`, `/*`, `#`, `<!--`). Multiple blocks per file and single-line blocks are supported.

## Known Limitations

- Single-line blocks hide the entire line.
- Comment suffix detection covers `*/` and `-->` only.
- If a model alters a placeholder's formatting, the script falls back to progressively looser matches.
- If a file is renamed or moved by a shell command, the moved file may retain placeholders; restore manually from the recovered backup.

## Requirements

- `jq`
- `shasum` or `sha1sum`
- Bash 3.2+

# sdd-cache helper scripts

Cross-session citation cache for [`source-driven-development`](../skills/source-driven-development/SKILL.md). The scripts skip redundant fetches without weakening the skill's "verify against current docs" guarantee.

These helpers are **not tied to any unsupported agent platform**. They are standalone shell scripts that can be wired into any target-tool wrapper that can provide compatible JSON payloads.

## Why

`source-driven-development` fetches official docs for framework-specific decisions. Across sessions, the same pages may be fetched repeatedly. This helper caches fetched content on disk, but **revalidates with the origin server on every reuse** via HTTP `If-None-Match` / `If-Modified-Since`.

Content is only served from cache when the server responds `304 Not Modified`, which is a fresh verification signal rather than blind trust in memory.

## Cache Location

By default, cache entries live in:

```text
.agent-skills/sdd-cache/
```

Override the project root with:

```bash
AGENT_SKILLS_PROJECT_DIR=/path/to/project
```

Add `.agent-skills/` to `.gitignore`.

## Payload Contract

The scripts read JSON from stdin.

### Pre-fetch payload

```json
{
  "tool_input": {
    "url": "https://example.com/docs",
    "prompt": "extract the API signature"
  }
}
```

### Post-fetch payload

```json
{
  "tool_input": {
    "url": "https://example.com/docs",
    "prompt": "extract the API signature"
  },
  "tool_response": "fetched or summarized content"
}
```

`tool_response` may also be an object with one of these fields: `result`, `output`, `text`, `content`, or `body`.

## How It Works

| Script | Action |
|---|---|
| `sdd-cache-pre.sh` | If an entry exists, sends a `HEAD` request with validators. On `304`, exits `2` and writes cached content to stderr. Otherwise exits `0`. |
| `sdd-cache-post.sh` | Captures the response body, records `ETag` / `Last-Modified`, and stores `{url, prompt, etag, last_modified, content, fetched_at}`. |

Freshness rules:

- entries are served only when the origin confirms `304 Not Modified`
- entries without `ETag` or `Last-Modified` are never cached
- cache key is `sha256(url)` truncated to 128 bits

## Local Testing

```bash
# Simulate a post-fetch payload: cache a page
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  },
  "tool_response": "useActionState(action, initialState) returns [state, formAction, isPending]"
}' | bash hooks/sdd-cache-post.sh

# Inspect cache
ls .agent-skills/sdd-cache/
cat .agent-skills/sdd-cache/*.json | jq .

# Simulate the next pre-fetch on the same URL
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  }
}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"
```

Expected:

- the post script creates one cache file only if the server provides validators
- the pre script exits `2` with cached content on stderr when the origin returns `304`, otherwise exits `0`

## Debugging

Enable debug logging with:

```bash
SDD_CACHE_DEBUG=1 bash hooks/sdd-cache-pre.sh
```

Or create a sentinel file:

```bash
mkdir -p .agent-skills/sdd-cache
touch .agent-skills/sdd-cache/.debug
```

Logs are written to `.agent-skills/sdd-cache/.debug.log`.

## Known Limitations

- **Body is prompt-shaped.** A cache hit returns the earlier agent's reading of the page. The original prompt is surfaced so the current agent can judge whether it applies.
- **Every cache write costs an extra HEAD request.** The post script re-queries the origin to capture validators.
- **Servers without validators are never cached.** Without `ETag` or `Last-Modified`, freshness cannot be verified.
- **Cache is local and per project.** There is no team-wide shared cache.

## Requirements

- `jq`
- `curl`
- `shasum` or `sha256sum`
- Bash 3.2+

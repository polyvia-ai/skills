# polyvia-ai/skills

Agent skills for [Polyvia AI platform](https://polyvia.ai)

## Install

**Claude Code / Cursor — one command installs skills + wires up the MCP server:**

```bash
/plugin add polyvia-ai/skills
```

**Skills only (all agent clients):**

```bash
npx skills add polyvia-ai/skills
```

Or install a specific skill:

```bash
npx skills add polyvia-ai/skills@polyvia
npx skills add polyvia-ai/skills@polyvia-mcp
```

> Set `POLYVIA_API_KEY=poly_...` in your environment before installing so the
> MCP server connects automatically. Get a key at
> [app.polyvia.ai → Settings → API](https://app.polyvia.ai/settings).

---

## What's included

### Plugin (`/plugin add`)

Installing via `/plugin add` bundles everything:

- ✅ Both skills loaded into context
- ✅ Polyvia MCP server configured at `https://app.polyvia.ai/mcp`
- ✅ All 10 Polyvia tools available immediately (ingest, query, groups, documents)

### Skills

#### `polyvia`

Use when writing code to ingest documents, query a knowledge base, or manage
document groups using the Python SDK.

```bash
pip install polyvia
export POLYVIA_API_KEY=poly_...
```

Covers: single and batch ingestion, status polling, natural-language queries
(scoped to document / group / workspace), group management, document updates
and deletion, usage monitoring.

#### `polyvia-mcp`

Use when connecting an AI client to the Polyvia hosted MCP server
(`https://app.polyvia.ai/mcp`). No code needed — the MCP server exposes all
10 Polyvia tools directly to Claude Desktop, the Anthropic beta MCP client,
OpenAI Responses API, or the OpenAI Agents SDK.

---

## Links

- [Polyvia Studio](https://app.polyvia.ai)
- [Docs](https://docs.polyvia.ai)
- [Python SDK](https://github.com/polyvia-ai/polyvia-python)
- [PyPI](https://pypi.org/project/polyvia)

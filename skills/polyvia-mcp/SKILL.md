---
name: polyvia-mcp
description: >
  Use this skill to connect an AI client (Claude Desktop, OpenAI Agents SDK,
  Anthropic beta MCP, or any MCP-compatible tool) to the Polyvia hosted MCP
  server. The MCP server exposes document ingestion, querying, and group
  management as tools that AI assistants can call directly — no manual
  tool-dispatch loop needed. Use this when setting up or configuring an MCP
  integration rather than writing Python SDK code.
license: MIT
metadata:
  author: polyvia-ai
  mcp-url: "https://app.polyvia.ai/mcp"
  docs: "https://docs.polyvia.ai/products/mcp"
allowed-tools: "Bash Read Write"
---

# Polyvia MCP Skill

The Polyvia MCP server is hosted at `https://app.polyvia.ai/mcp` (streamable HTTP transport). Connect once and any MCP-compatible AI client can ingest, search, and query documents without any code.

Requires a `poly_...` API key from **app.polyvia.ai → Settings → API**.

---

## Claude Desktop

Add to `~/.claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "polyvia": {
      "type": "http",
      "url": "https://app.polyvia.ai/mcp",
      "headers": {
        "Authorization": "Bearer poly_<your-key>"
      }
    }
  }
}
```

Restart Claude Desktop. **Polyvia** will appear in the MCP server list.

Or generate the snippet programmatically:

```python
from polyvia import Polyvia
Polyvia(api_key="poly_...").mcp.print_claude_desktop_snippet()
```

---

## Anthropic beta MCP client

```python
from anthropic import Anthropic
from polyvia import Polyvia

polyvia = Polyvia(api_key="poly_...")
ant     = Anthropic()

response = ant.beta.messages.create(
    model="claude-opus-4-5",
    max_tokens=1000,
    messages=[{"role": "user", "content": "What documents do I have?"}],
    mcp_servers=[polyvia.mcp.to_anthropic_mcp_server()],
    betas=["mcp-client-2025-04-04"],
)
print(response.content[0].text)
```

---

## OpenAI Responses API

```python
from openai import OpenAI
from polyvia import Polyvia

response = OpenAI().responses.create(
    model="gpt-4o",
    tools=[Polyvia(api_key="poly_...").mcp.to_openai_responses_tool()],
    input="What are my Q4 findings?",
)
print(response.output_text)
```

---

## OpenAI Agents SDK

```python
from agents import Agent, Runner
from agents.mcp import MCPServerStreamableHTTP
from polyvia import Polyvia

cfg    = Polyvia(api_key="poly_...").mcp.to_openai_mcp_server()
server = MCPServerStreamableHTTP(url=cfg["url"], headers=cfg["headers"])
agent  = Agent(name="Research", mcp_servers=[server])
result = Runner.run_sync(agent, "Summarise my Finance documents.")
print(result.final_output)
```

---

## Available MCP Tools

| Tool | What it does |
|------|-------------|
| `polyvia_ingest_document` | Upload from URL or base64 content |
| `polyvia_check_ingestion_status` | Poll a parse task |
| `polyvia_list_documents` | List documents, filter by status |
| `polyvia_get_document` | Get metadata + summary for one document |
| `polyvia_query` | Ask a question — scoped to document, group, or workspace |
| `polyvia_list_groups` | List all groups |
| `polyvia_create_group` | Create a new group |
| `polyvia_update_document` | Move document to a group / remove from group |
| `polyvia_delete_document` | Delete a document |
| `polyvia_delete_group` | Delete a group (optionally wipe its documents first) |

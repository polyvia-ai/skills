# Polyvia API Cheat Sheet

Base URL: `https://app.polyvia.ai`
Auth: `Authorization: Bearer poly_<key>`

## Ingest

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v1/ingest` | Upload one file (`multipart/form-data`: `file`, `name?`, `group_id?`) |
| POST | `/api/v1/ingest/batch` | Upload multiple files (`files[]`, `names[]?`, `group_id?`) |
| GET  | `/api/v1/ingest/{task_id}` | Poll status → `pending` / `parsing` / `completed` / `failed` |

## Documents

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET    | `/api/v1/documents` | List (`status?`, `group_id?`, `group_ids?`) |
| GET    | `/api/v1/documents/{id}` | Get one |
| PATCH  | `/api/v1/documents/{id}` | Update group (`{"group_id": "g_..." \| null}`) |
| DELETE | `/api/v1/documents/{id}` | Delete |

## Groups

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET    | `/api/v1/groups` | List all groups |
| POST   | `/api/v1/groups` | Create (`{"name": "..."}`) → `{"group_id": "..."}` |
| DELETE | `/api/v1/groups/{id}/documents` | Remove all documents from group |
| DELETE | `/api/v1/groups/{id}` | Delete group (must be empty first) |

## Query

| Method | Endpoint | Body |
|--------|----------|------|
| POST | `/api/v1/query` | `{"query": "...", "document_id"?: "...", "group_id"?: "...", "group_ids"?: [...]}` |

## Usage

| Method | Endpoint | Returns |
|--------|----------|---------|
| GET | `/api/v1/usage` | Per-key counters (`requests`, `ingests`, `queries`) + per-workspace totals (`pages`, `audio_seconds`, `documents_stored`) — each as `{period, total}` |
| GET | `/api/v1/rate-limits` | Plan limits, remaining capacity, reset timestamps |

## Python SDK Quick Reference

```python
from polyvia import Polyvia, AsyncPolyvia
client = Polyvia()  # POLYVIA_API_KEY env var

# Ingest  (group= takes a group NAME, created if missing; group_id= is the raw id)
client.ingest.file(path, name=None, group=None, group_id=None)     # → IngestResult
client.ingest.batch(paths, names=None, group=None, group_id=None)  # → list[BatchIngestItem]
client.ingest.status(task_id)                             # → IngestionStatus
client.ingest.wait(task_id, poll_interval=5, timeout=300)

# Query  (group= = name, must already exist; group_ids= = list of ids)
client.query(q, document_id=None, group=None, group_id=None, group_ids=None)  # → QueryResult

# Documents
client.documents.list(status=None, group_id=None, group_ids=None)
client.documents.get(document_id)
client.documents.update(document_id, group_id=...)
client.documents.delete(document_id)

# Groups
client.groups.list()
client.groups.get_or_create(name)                 # → Group  (idempotent; prefer this)
client.groups.find(name)                          # → Group | None
client.groups.create(name)                        # → {"group_id": "..."}  (always new)
client.groups.delete(group_id, delete_documents=False)
client.groups.delete_documents(group_id)

# Usage
client.usage()        # → Usage
client.rate_limits()  # → RateLimits

# MCP & Agent Tools
client.mcp.to_anthropic_mcp_server()
client.mcp.to_openai_responses_tool()
client.mcp.to_claude_desktop_config()
client.tools.anthropic()   # → (tools, call_fn)
client.tools.openai()      # → (tools, call_fn)
client.tools.langchain()   # → list[StructuredTool]
```

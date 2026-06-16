---
name: polyvia
description: >
  Use this skill whenever you need to ingest documents into a knowledge base,
  query documents with natural language, manage document groups, or build
  document-aware agents using the Polyvia API. Covers uploading single or
  multiple files, polling ingestion status, querying across all documents or
  scoped to a group, creating/deleting groups, and checking workspace usage.
  Requires a POLYVIA_API_KEY environment variable (keys start with poly_).
license: MIT
metadata:
  author: polyvia-ai
  sdk: "pip install polyvia"
  docs: "https://docs.polyvia.ai"
allowed-tools: "Bash Read Write"
---

# Polyvia Skill

Polyvia is multimodal document intelligence at scale for AI agents. You can ingest documents (`.pdf`, `.docx`, `.pptx`, `.txt`, `.md`), images (`.png`, `.jpg`, `.webp`, `.gif`), and audio (`.wav`, `.mp3`, `.m4a`), wait for them to be parsed and indexed, then query them with natural language — scoped to a single document, a group, or the entire workspace.

## Setup

Install the SDK and set the API key:

```bash
pip install polyvia
export POLYVIA_API_KEY=poly_...   # or pass api_key= explicitly
```

Get an API key at **app.polyvia.ai → API** (in the sidebar).

---

## Core Workflows

### Ingest a single document and query it

```python
from polyvia import Polyvia

client = Polyvia()  # reads POLYVIA_API_KEY from env

result = client.ingest.file("report.pdf", name="Q4 Report")
client.ingest.wait(result.task_id)   # blocks until indexed (raises on failure)

answer = client.query("What are the key findings?", document_id=result.document_id)
print(answer.answer)
```

### Ingest multiple documents at once

```python
batch = client.ingest.batch(
    ["q3.pdf", "q4.pdf"],
    names=["Q3 Report", "Q4 Report"],
    group="Finance",   # optional — group name (created if it doesn't exist yet)
)
for item in batch:
    client.ingest.wait(item.task_id)
```

### Query across the whole workspace

```python
answer = client.query("What risks are mentioned across all reports?")
print(answer.answer)
```

### Query scoped to a group

```python
answer = client.query("Compare the key findings.", group="Finance")
# or multiple groups (by id):
answer = client.query("Compare Q3 vs Q4.", group_ids=["g_q3", "g_q4"])
```

---

## Groups

```python
# Groups have a human name + an opaque backend id. Pass the name straight to
# ingest/query (above); use get_or_create when you want the Group object/id.
group = client.groups.get_or_create("Finance")   # idempotent — never duplicates
group.id, group.name

# List
for g in client.groups.list():
    print(g.id, g.name)

# Delete (documents first, then group)
client.groups.delete(group.id, delete_documents=True)
```

---

## Document Management

```python
# List — filter by status and/or group
docs = client.documents.list(status="completed", group_id="g_...")

# Move to a different group / remove from group
client.documents.update("doc_...", group_id="g_other")
client.documents.update("doc_...", group_id=None)

# Delete
client.documents.delete("doc_...")
```

---

## Monitoring Usage

```python
usage  = client.usage()
limits = client.rate_limits()

print(usage.usage.requests.period)            # API requests this month (per key)
print(usage.usage.pages.period)               # pages processed this month (workspace)
print(usage.usage.audio_seconds.period / 60)  # minutes of audio this month
print(usage.usage.documents_stored)           # live document count
print(limits.current["remaining_this_minute"])
```

---

## Async Client

Use `AsyncPolyvia` for async/await code — identical API surface:

```python
import asyncio
from polyvia import AsyncPolyvia

async def main():
    async with AsyncPolyvia() as client:
        result = await client.ingest.file("report.pdf")
        await client.ingest.wait(result.task_id)
        answer = await client.query("Key findings?")
        print(answer.answer)

asyncio.run(main())
```

---

## Error Handling

```python
from polyvia import (
    AuthenticationError,  # 401 — invalid key
    NotFoundError,         # 404 — document/group/task not found
    RateLimitError,        # 429 — slow down
    IngestionError,        # document parsing failed
    IngestionTimeout,      # ingest.wait() timed out
)
```

---

## Common Patterns

**Check if a document is ready before querying:**
```python
status = client.ingest.status(task_id)
if status.status == "completed":
    answer = client.query("...", document_id=status.document_id)
```

**Ingest from a URL (via MCP):** Use the MCP server tool `polyvia_ingest_document` with `source_url=...` — the MCP server handles remote fetching.

**Build an agent with tools:** `client.tools.anthropic()` and `client.tools.openai()` return tool schemas and an executor for use in a manual tool-dispatch loop. See `pip install polyvia` → [docs.polyvia.ai](https://docs.polyvia.ai/products/python-sdk).

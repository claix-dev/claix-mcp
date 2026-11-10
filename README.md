# Claix MCP Server

> Hosted MCP server for **Claix**, the security layer between your SQL database and AI agents.

Connect Cursor, Claude, Windsurf, n8n, Make, or any MCP client to Claix over Streamable HTTP (and legacy SSE). Primary products on this server: **Claix Retrieve** (controlled reads) and **Claix Guard** (allow/deny on proposed writes). **Claix Ingest** (documents → JSON/Markdown) is secondary.

- **Registry:** [`dev.claix/claix`](https://registry.modelcontextprotocol.io/)
- **Endpoint:** https://www.claix.dev/mcp
- **Transport:** Streamable HTTP and legacy SSE
- **MCP version:** 1.14.0
- **Tools:** 33
- **Docs:** https://www.claix.dev/documentation/mcp
- **Server card:** https://www.claix.dev/.well-known/mcp/server-card.json
- **Smithery:** https://smithery.ai/server/info-f4xz/claix

[![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-MCP-7C3AED?style=flat-square)](https://modelcontextprotocol.io/)
[![Claix](https://img.shields.io/badge/Claix-Security%20layer-00C853?style=flat-square)](https://www.claix.dev)
[![Tools](https://img.shields.io/badge/Tools-33-00C853?style=flat-square)](https://www.claix.dev/documentation/mcp)
[![Registry](https://img.shields.io/badge/MCP%20Registry-dev.claix%2Fclaix-111827?style=flat-square)](https://registry.modelcontextprotocol.io/)
[![Smithery](https://img.shields.io/badge/Smithery-Install-111827?style=flat-square)](https://smithery.ai/server/info-f4xz/claix)

## What Claix is

Claix sits between AI agents and your SQL database (PostgreSQL, MySQL, MariaDB, and Postgres-compatible hosts such as Supabase).

| Product | Job | MCP |
| --- | --- | --- |
| **Retrieve** | Controlled reads: routes whitelist tables/columns; agents answer only inside that allow-list | `claix.routes.*`, `claix.agents.*`, `claix.retrieve.ask` |
| **Guard** | Before your app runs an `INSERT` / `UPDATE` / `DELETE`, Claix answers **allow** or **deny** with a reason. Claix never executes the statement | `claix.guard.check` → `POST /check-sql` |
| **Ingest** (secondary) | Documents → schema-validated JSON or Markdown; optional multi-document Q&A | `claix.schemas.*`, `claix.extract.*`, `claix.agent.*`, `claix.markdown.*`, `claix.multi_document.ask` |

Pay per use. Auth with `x-api-key`.

## What you can do

### Claix Retrieve

Create routes and agents, then ask. The agent reads only the tables and columns on its routes and returns raw rows (`bruto`) or typed answers (`razonada`). Every ask is traced (`log_id`).

```text
claix.routes.create
        ↓
claix.agents.create
        ↓
claix.retrieve.ask
        ↓
Typed answers from the live database
```

### Claix Guard

Your agent proposes a write. Your executor calls Guard first. Claix decides; your code runs the SQL only when `allowed` is `true`. A denial is still HTTP 200 — always read `allowed`.

```text
Agent proposes INSERT / UPDATE / DELETE
        ↓
claix.guard.check   (sql, params?, connection_id?)
        ↓
{ allowed, reason, decision_id, … }
        ↓
Your app executes only if allowed === true
```

### Claix Ingest (secondary)

List or create a schema, then extract a file to JSON, convert it to Markdown, or ask up to 5 typed questions across up to 10 files. When a schema has Save Data on (`window_context`), the mapped row can be inserted into the connected database and the response can include `row_id`.

```text
claix.schemas.list
        ↓
claix.extract.pdf
        ↓
Structured JSON, and row_id when Save Data is on
```

## Quick start

### 1. Get an API key

Create an account at [claix.dev](https://www.claix.dev) and generate an API key.

```http
x-api-key: YOUR_CLAIX_API_KEY
```

> [!IMPORTANT]
> Do not put API keys in public repositories, screenshots, browser code, or commit history.

### 2. Connect from Cursor

Open **Cursor → Settings → MCP** and add:

```json
{
  "mcpServers": {
    "claix": {
      "url": "https://www.claix.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_CLAIX_API_KEY"
      }
    }
  }
}
```

Or install from the [official MCP Registry](https://registry.modelcontextprotocol.io/) as `dev.claix/claix` (remote URL `https://www.claix.dev/mcp`, header `x-api-key`).

### 3. Ask a Retrieve agent

After the route and the agent exist (dashboard or MCP tools), call `claix.retrieve.ask`:

```json
{
  "agent_name": "restaurant-ops",
  "questions": [
    {
      "question": "How many tables are reserved this Saturday between 20:00 and 22:00?",
      "format": "int"
    }
  ]
}
```

`format` is required when the agent's `response_mode` is `razonada`. It is optional when the mode is `bruto`. Formats: `string`, `int`, `boolean`, `timestamp`, `timestamptz`, `date`, `json`, `array:string`, `array:int`. Up to 5 questions per call.

This is `POST /retrieve/{agent_name}`.

### 4. Check a write with Guard

Before your app executes a change:

```json
{
  "sql": "INSERT INTO shop.orders (customer_id, total) VALUES ($1, $2)",
  "params": ["c-1042", 6200],
  "connection_id": "8f14e45f-ceea-4e6f-8b23-1e2d3c4b5a6f"
}
```

Call `claix.guard.check`. Read `allowed`, `reason`, and `decision_id`. Claix does not run the SQL.

### 5. Extract a PDF (optional Ingest)

Call `claix.schemas.list`, then `claix.extract.pdf` with the `schema_id` and the file:

```json
{
  "schema_id": "YOUR_SCHEMA_ID",
  "file_base64": "JVBERi0xLjQK..."
}
```

## Install with Smithery

```bash
npx -y @smithery/cli run info-f4xz/claix
```

```json
{
  "mcpUrl": "https://www.claix.dev/mcp",
  "headers": {
    "x-api-key": "YOUR_CLAIX_API_KEY"
  }
}
```

## Connect with Claude Desktop

Claude Desktop can connect through `mcp-remote`:

```json
{
  "mcpServers": {
    "claix": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://www.claix.dev/mcp",
        "--header",
        "x-api-key:YOUR_CLAIX_API_KEY"
      ]
    }
  }
}
```

## Other MCP clients

Any client that can send a custom header to a remote MCP server can use:

```json
{
  "url": "https://www.claix.dev/mcp",
  "headers": {
    "x-api-key": "YOUR_CLAIX_API_KEY"
  }
}
```

That includes Cursor, Claude Desktop, Claude Code, Windsurf, Cline, Continue, Zed, n8n, Make, and LangChain or LangGraph clients that speak Streamable HTTP or SSE.

Tools also accept an optional `api_key` argument when the client cannot send headers. Prefer the header.

## Authentication

```http
POST /mcp HTTP/1.1
Host: www.claix.dev
Content-Type: application/json
Accept: application/json, text/event-stream
x-api-key: YOUR_CLAIX_API_KEY
```

## Transport

| Mode | Typical use | Connection |
| --- | --- | --- |
| Streamable HTTP | Cursor, Smithery, n8n, Make, current MCP clients | `POST https://www.claix.dev/mcp` |
| Legacy SSE | Claude Desktop via `mcp-remote`, older session clients | `GET /mcp`, then `POST /mcp/message?sessionId=...` |

Both modes use the same base URL. `claix.dev` redirects to `www.claix.dev`.

## Tools

| Category | Tools | Purpose |
| --- | --- | --- |
| **Retrieve** | `claix.retrieve.ask` | Ask an agent. `POST /retrieve/{agent_name}` |
| **Routes** | `claix.routes.create`, `claix.routes.edit`, `claix.routes.delete` | Whitelist tables/columns and route rules |
| **Agents** | `claix.agents.create`, `claix.agents.edit`, `claix.agents.pause` | Create/edit agents or set `paused` / `active` |
| **Guard** | `claix.guard.check` | Allow/deny a proposed write. `POST /check-sql`. Does not execute SQL |
| Schemas (Ingest) | `claix.schemas.list`, `get`, `create`, `update`, `delete` | Extraction schemas, including Save Data (`window_context`) |
| Extraction (Ingest) | `claix.extract.excel`, `pdf`, `doc`, `image`, `text`, `audio` | File or text → schema-validated JSON |
| Agent mode (Ingest) | `claix.agent.excel`, `pdf`, `doc`, `image`, `text`, `audio` | Same inputs when the schema has agent mode on |
| Markdown (Ingest) | `claix.markdown.excel`, `pdf`, `doc`, `image`, `text`, `audio` | File → Markdown. Optional `save=true` can return `row_id` |
| Multi-document | `claix.multi_document.ask` | Up to 10 files and 5 typed questions. `POST /multi-document` |
| Convert | `claix.convert.json_to_excel` | JSON → `.xlsx` |

Do not invent `schema_id`, `route_id`, `agent_id`, or `connection_id`. List or create them first.

Retrieve, Guard checks, extraction, agent mode, and multi-document return tracing fields (`log_id` / `decision_id` as applicable).

## Workflows

### Ask the database (Retrieve)

```text
claix.routes.create
→ claix.agents.create
→ claix.retrieve.ask
```

Or skip the first two when the agent already exists in the dashboard, and call `claix.retrieve.ask` with its slug.

### Check a write (Guard)

```text
claix.guard.check
→ read allowed / reason
→ your executor runs SQL only if allowed
```

### Extract a PDF (Ingest, secondary)

```text
claix.schemas.list
→ claix.extract.pdf
```

### Ask across several files

```text
claix.multi_document.ask
```

## Prompts

`prompts/list` includes:

- `workflow.retrieve-ask`
- `workflow.guard-check`
- `workflow.discover-and-extract`
- `workflow.agent-document-analysis`
- `workflow.invoice-pdf`
- `workflow.markdown-save`
- `workflow.multi-document-ask`

## Resources

- MCP documentation: `claix://docs/mcp`
- OpenAPI specification: `claix://docs/openapi`
- Tool catalog: `claix://docs/tools`
- Server card: https://www.claix.dev/.well-known/mcp/server-card.json
- Product guide: https://github.com/claix-dev/claix-for-devs
- Full documentation: https://www.claix.dev/documentation/mcp

## Security

- Store the API key in the client's secret storage or in an environment variable.
- Prefer the `x-api-key` header over the per-tool `api_key` argument.
- `claix.schemas.delete`, `claix.routes.delete`, and pausing an agent change the account. Confirm before you call them.
- Retrieve uses the read-only database role you connected.
- Guard only checks proposed writes. Your executor holds write credentials and must respect `allowed`.

## Official registry

Published as **`dev.claix/claix`** on the [MCP Registry](https://registry.modelcontextprotocol.io/).

```bash
curl "https://registry.modelcontextprotocol.io/v0.1/servers?search=dev.claix/claix"
```

## Links

- Website: https://www.claix.dev
- MCP documentation: https://www.claix.dev/documentation/mcp
- Retrieve: https://www.claix.dev/documentation/call-an-agent
- Guard: https://www.claix.dev/documentation/check-sql
- Server card: https://www.claix.dev/.well-known/mcp/server-card.json
- Smithery: https://smithery.ai/server/info-f4xz/claix
- This repo: https://github.com/claix-dev/claix-mcp

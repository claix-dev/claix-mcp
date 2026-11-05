# Claix MCP Server

> The hosted MCP server for Claix, the security layer between your SQL database and AI agents.

Connect Cursor, Claude, Windsurf, n8n, Make, or any MCP client to Claix. The server exposes **32 tools** (v1.13.0). Retrieve tools ask your live database through the routes and agents you defined. The rest turn documents into structured JSON or Markdown.

Guard, the check on `INSERT`, `UPDATE`, and `DELETE`, is `POST /check-sql` on the HTTP API. It is not one of these MCP tools. Docs: [claix.dev/documentation/check-sql](https://claix.dev/documentation/check-sql).

- **Endpoint:** https://claix.dev/mcp
- **Transport:** Streamable HTTP and legacy SSE
- **MCP version:** 1.13.0
- **Tools:** 32
- **Docs:** https://claix.dev/documentation/mcp
- **Server card:** https://claix.dev/.well-known/mcp/server-card.json
- **Smithery:** https://smithery.ai/server/info-f4xz/claix

[![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-MCP-7C3AED?style=flat-square)](https://modelcontextprotocol.io/)
[![Claix](https://img.shields.io/badge/Claix-Security%20layer-00C853?style=flat-square)](https://claix.dev)
[![Tools](https://img.shields.io/badge/Tools-32-00C853?style=flat-square)](https://claix.dev/documentation/mcp)
[![Smithery](https://img.shields.io/badge/Smithery-Install-111827?style=flat-square)](https://smithery.ai/server/info-f4xz/claix)

## What you can do

**Claix Retrieve.** Create routes and agents, then ask. The agent reads only the tables and columns on its routes and returns raw rows or typed answers. Supported databases: PostgreSQL, MySQL, MariaDB, and Postgres-compatible databases such as Supabase.

```text
claix.routes.create
        ↓
claix.agents.create
        ↓
claix.retrieve.ask
        ↓
Typed answers from the live database
```

**Claix Ingest.** List or create a schema, then extract a file to JSON, convert it to Markdown, or ask up to 5 typed questions across up to 10 files. When a schema has Save Data on (`window_context`), the mapped row can be inserted into the connected database and the response can include `row_id`.

```text
claix.schemas.list
        ↓
claix.extract.pdf
        ↓
Structured JSON, and row_id when Save Data is on
```

## Quick start

### 1. Get an API key

Create an account at [claix.dev](https://claix.dev) and generate an API key.

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
      "url": "https://claix.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_CLAIX_API_KEY"
      }
    }
  }
}
```

### 3. Ask a Retrieve agent

After the route and the agent exist, call `claix.retrieve.ask`:

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

### 4. Extract a PDF

Call `claix.schemas.list`, then `claix.extract.pdf` with the `schema_id` and the file:

```json
{
  "schema_id": "YOUR_SCHEMA_ID",
  "file_base64": "JVBERi0xLjQK..."
}
```

The structured fields come back on the tool result. If Save Data is on, the response can also include `row_id`.

## Install with Smithery

```bash
npx -y @smithery/cli run info-f4xz/claix
```

```json
{
  "mcpUrl": "https://claix.dev/mcp",
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
        "https://claix.dev/mcp",
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
  "url": "https://claix.dev/mcp",
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
Host: claix.dev
Content-Type: application/json
Accept: application/json, text/event-stream
x-api-key: YOUR_CLAIX_API_KEY
```

## Transport

| Mode | Typical use | Connection |
| --- | --- | --- |
| Streamable HTTP | Cursor, Smithery, n8n, Make, current MCP clients | `POST https://claix.dev/mcp` |
| Legacy SSE | Claude Desktop via `mcp-remote`, older session clients | `GET /mcp`, then `POST /mcp/message?sessionId=...` |

Both modes use the same base URL.

## Tools

| Category | Tools | Purpose |
| --- | --- | --- |
| Retrieve | `claix.retrieve.ask` | Ask an agent. `POST /retrieve/{agent_name}` |
| Routes | `claix.routes.create`, `claix.routes.edit`, `claix.routes.delete` | Define which tables and columns a question may use |
| Agents | `claix.agents.create`, `claix.agents.edit`, `claix.agents.pause` | Create the agent, edit it, or set `paused` / `active` |
| Schemas | `claix.schemas.list`, `get`, `create`, `update`, `delete` | Extraction schemas, including Save Data (`window_context`) |
| Extraction | `claix.extract.excel`, `pdf`, `doc`, `image`, `text`, `audio` | File or text to schema-validated JSON |
| Agent mode | `claix.agent.excel`, `pdf`, `doc`, `image`, `text`, `audio` | The same inputs when the schema has agent mode on |
| Markdown | `claix.markdown.excel`, `pdf`, `doc`, `image`, `text`, `audio` | File to Markdown. Optional `save=true` can return `row_id` |
| Multi-document | `claix.multi_document.ask` | Up to 10 files and 5 typed questions. `POST /multi-document` |
| Convert | `claix.convert.json_to_excel` | JSON to `.xlsx` |

Do not invent `schema_id`, `route_id`, or `agent_id`. List or create them first.

Extraction, agent mode, multi-document, and retrieve return a `log_id`.

## Workflows

### Ask the database

```text
claix.routes.create
→ claix.agents.create
→ claix.retrieve.ask
```

Or skip the first two when the agent already exists in the dashboard, and call `claix.retrieve.ask` with its slug.

### Extract a PDF

```text
claix.schemas.list
→ claix.extract.pdf
```

### Ask across several files

```text
claix.multi_document.ask
```

### Run agent mode on a document

```text
claix.schemas.list
→ schema with is_agent_mode = true
→ claix.agent.pdf
```

## Prompts

`prompts/list` includes:

- `workflow.retrieve-ask`
- `workflow.discover-and-extract`
- `workflow.agent-document-analysis`
- `workflow.invoice-pdf`
- `workflow.markdown-save`
- `workflow.multi-document-ask`

## Resources

- MCP documentation: `claix://docs/mcp`
- OpenAPI specification: `claix://docs/openapi`
- Tool catalog: `claix://docs/tools`
- Server card: https://claix.dev/.well-known/mcp/server-card.json
- Product guide: https://github.com/claix-dev/claix-for-devs
- Full documentation: https://claix.dev/documentation/mcp

## Security

- Store the API key in the client's secret storage or in an environment variable.
- Prefer the `x-api-key` header over the per-tool `api_key` argument.
- `claix.schemas.delete`, `claix.routes.delete`, and pausing an agent change the account. Confirm before you call them.
- Retrieve uses the read-only database role you connected. Guard, on `POST /check-sql`, is the check for writes, and it does not execute them.

## Links

- Website: https://claix.dev
- MCP documentation: https://claix.dev/documentation/mcp
- Retrieve: https://claix.dev/documentation/call-an-agent
- Guard: https://claix.dev/documentation/check-sql
- Server card: https://claix.dev/.well-known/mcp/server-card.json
- Smithery: https://smithery.ai/server/info-f4xz/claix

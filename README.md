# Claix MCP Server

> Turn documents into structured, source-cited knowledge for AI agents.

Claix is a hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Claude Code, Windsurf, n8n, Make and custom agents secure access to documents, extracted data and persistent knowledge.

Instead of building custom document pipelines, connect your agent to Claix and use 23 tools for extraction, document analysis, persistent context and Knowledge Spaces.

- **Endpoint:** https://claix.dev/mcp
- **Transport:** Streamable HTTP and legacy SSE
- **MCP version:** 1.11.0
- **Tools:** 23
- **Docs:** https://claix.dev/documentation/mcp
- **Smithery:** https://smithery.ai/server/info-f4xz/claix

[![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-MCP-7C3AED?style=flat-square)](https://modelcontextprotocol.io/)
[![Claix](https://img.shields.io/badge/Claix-Document%20Intelligence-00C853?style=flat-square)](https://claix.dev)
[![Tools](https://img.shields.io/badge/Tools-23-00C853?style=flat-square)](https://claix.dev/documentation/mcp)
[![Smithery](https://img.shields.io/badge/Smithery-Install-111827?style=flat-square)](https://smithery.ai/server/info-f4xz/claix)

## What it does

Claix lets an AI agent:

- Extract structured JSON from PDFs, Excel/CSV, Word, text, HTML, XML and images.
- Analyze documents with Agent Mode.
- Persist processed documents and keep their context available.
- Ask follow-up questions without re-uploading the file.
- Group related documents into Knowledge Spaces.
- Query multiple documents together.
- Return typed results with optional source verification.

Example flow:

```text
PDF, Excel, image or text
        ↓
Claix MCP Server
        ↓
Structured JSON + document_id
        ↓
Persistent context and follow-up questions
```

## Quick start

### 1. Get a Claix API key

Create an account at [claix.dev](https://claix.dev) and generate an API key.

Claix authenticates MCP requests with:

```http
x-api-key: YOUR_CLAIX_API_KEY
```

> [!IMPORTANT]
> Do not expose API keys in public repositories, screenshots, browser code or commit history.

### 2. Connect from Cursor

Open **Cursor → Settings → MCP** and add:

```json
{
  "mcpServers": {
    "claix": {
      "url": "[https://claix.dev/mcp](https://claix.dev/mcp)",
      "headers": {
        "x-api-key": "YOUR_CLAIX_API_KEY"
      }
    }
  }
}
```

### 3. List your schemas

Call:

```text
claix.schemas.list
```

This returns the schemas available in your workspace, including their `schema_id`, document type and whether Agent Mode is enabled.

### 4. Extract a PDF

Call:

```text
claix.extract.pdf
```

With arguments similar to:

```json
{
  "schema_id": "YOUR_SCHEMA_ID",
  "file_base64": "JVBERi0xLjQK..."
}
```

Claix returns structured data through:

```text
structuredContent.data
```

Example:

```json
{
  "success": true,
  "data": {
    "invoice_number": "INV-2026-0841",
    "supplier_name": "Northwind Supplies",
    "total_amount": 1284.50,
    "currency": "EUR"
  }
}
```

### 5. Keep the document context

If your schema has Context Window enabled, the extraction response includes a `document_id`.

Use it to ask follow-up questions:

```text
claix.window_context.ask
```

Or retrieve the stored document content:

```text
claix.window_context.get
```

You can also group documents using Knowledge Spaces and query all of them together:

```text
claix.spaces.create
claix.spaces.add_document
claix.space_context.ask
```

## Install with Smithery

One-command install:

```bash
npx -y @smithery/cli run info-f4xz/claix
```

Manual configuration:

```json
{
  "mcpUrl": "[https://claix.dev/mcp](https://claix.dev/mcp)",
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
        "[https://claix.dev/mcp](https://claix.dev/mcp)",
        "--header",
        "x-api-key:YOUR_CLAIX_API_KEY"
      ]
    }
  }
}
```

## Connect with other MCP clients

For remote MCP clients that support custom headers, use:

```json
{
  "url": "[https://claix.dev/mcp](https://claix.dev/mcp)",
  "headers": {
    "x-api-key": "YOUR_CLAIX_API_KEY"
  }
}
```

Compatible clients include Cursor, Claude Desktop, Claude Code, Windsurf, Cline, Continue, Zed, Lovable, Replit Agent, n8n, Make, LangChain/LangGraph and any MCP client supporting Streamable HTTP or SSE.

## Authentication

The recommended method is the `x-api-key` HTTP header:

```http
POST /mcp HTTP/1.1
Host: claix.dev
Content-Type: application/json
Accept: application/json, text/event-stream
x-api-key: YOUR_CLAIX_API_KEY
```

For clients that cannot send custom headers, supported tools also accept an optional `api_key` argument.

## Transport modes

| Mode | Typical use | Connection |
|---|---|---|
| Streamable HTTP | Cursor, Smithery, modern MCP clients, n8n, Make | `POST https://claix.dev/mcp` |
| Legacy SSE | Claude Desktop via `mcp-remote`, legacy session clients | `GET /mcp` + `POST /mcp/message?sessionId=...` |

Both modes are available at the same base URL.

## Tool categories

| Category | Tools | Purpose |
|---|---|---|
| Schemas | `claix.schemas.*` | Create, list and delete extraction schemas |
| Extraction | `claix.extract.*` | Convert files or text into structured JSON |
| Agent Mode | `claix.agent.*` | Flexible document analysis and agent-oriented output |
| Document context | `claix.window_context.*` | Retrieve or query a persisted document |
| Knowledge Spaces | `claix.spaces.*`, `claix.space_context.ask` | Group and query multiple documents |
| Document management | `claix.document.*` | Delete or replace persisted documents |
| Utilities | `claix.convert.json_to_excel` | Convert JSON into an Excel file |

## Main workflows

### Extract a PDF

```text
claix.schemas.list
→ claix.extract.pdf
→ structuredContent.data
```

### Ask questions about a document

```text
claix.extract.pdf
→ document_id
→ claix.window_context.ask
```

### Query multiple documents

```text
claix.spaces.create
→ claix.extract.pdf with space_id
→ claix.space_context.ask
```

### Run Agent Mode

```text
claix.schemas.list
→ find schema with is_agent_mode=true
→ claix.agent.pdf / claix.agent.excel / claix.agent.doc
```

## Reusable prompts

Claix exposes workflow prompts through `prompts/list`:

- `workflow.discover-and-extract`
- `workflow.agent-document-analysis`
- `workflow.invoice-pdf`
- `workflow.window-context-ask`
- `workflow.space-context-ask`

## Resources

- MCP documentation: `claix://docs/mcp`
- OpenAPI specification: `claix://docs/openapi`
- Tool catalog: `claix://docs/tools`
- Server card: https://claix.dev/.well-known/mcp/server-card.json
- Full documentation: https://claix.dev/documentation/mcp

## Security

- Use environment variables or your client’s secure secrets storage for API keys.
- Prefer header authentication over the per-tool `api_key` fallback.
- `claix.document.delete` and `claix.spaces.delete` are destructive and expose `destructiveHint`.
- Document deletion is irreversible.

## Links

- Website: https://claix.dev
- MCP documentation: https://claix.dev/documentation/mcp
- Server card: https://claix.dev/.well-known/mcp/server-card.json
- Smithery listing: https://smithery.ai/server/info-f4xz/claix
- Official MCP Registry: https://registry.modelcontextprotocol.io

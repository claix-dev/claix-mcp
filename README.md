# Claix MCP Server

> Give AI assistants, coding agents, and automation workflows document intelligence through MCP: extract structured JSON from files, persist document context, query documents, and reason across Knowledge Spaces.

[![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-MCP-7C3AED?style=flat-square)](https://modelcontextprotocol.io/)
[![Claix](https://img.shields.io/badge/Claix-Document%20Intelligence-00C853?style=flat-square)](https://claix.dev)
[![Transport](https://img.shields.io/badge/Transport-Streamable%20HTTP%20%2B%20SSE-0EA5E9?style=flat-square)](https://claix.dev/mcp)
[![Tools](https://img.shields.io/badge/Tools-20-00C853?style=flat-square)](https://claix.dev/documentation/mcp)
[![Smithery](https://img.shields.io/badge/Smithery-Install-111827?style=flat-square)](https://smithery.ai/server/info-f4xz/claix)

Claix is a remote **Model Context Protocol (MCP) server** for document intelligence. It enables MCP-capable clients to process PDFs, Excel and CSV files, Word and text documents, images, HTML, XML, and plain text without building custom REST integrations.

The server exposes structured extraction, Agent Mode, schema management, persistent document context, Knowledge Spaces, reusable workflow prompts, and documentation resources through a single hosted endpoint.

```text
Document or raw content
        ↓
Claix MCP Server
        ↓
Schema-validated structured JSON
        ↓
AI assistant, coding agent, backend, workflow, CRM, ERP, or database
```

## Endpoint

```text
[https://claix.dev/mcp](https://claix.dev/mcp)
```

### MCP server metadata

```text
[https://claix.dev/.well-known/mcp/server-card.json](https://claix.dev/.well-known/mcp/server-card.json)
```

### Official documentation

```text
[https://claix.dev/documentation/mcp](https://claix.dev/documentation/mcp)
```

### Smithery listing

```text
[https://smithery.ai/server/info-f4xz/claix](https://smithery.ai/server/info-f4xz/claix)
```

## What Claix provides

- Extract schema-validated JSON from PDFs, Excel/CSV files, documents, images, text, HTML, and XML.
- Use Agent Mode for more flexible document analysis when enabled on a schema.
- Create, list, and delete extraction schemas.
- Persist processed documents and retrieve a `document_id`.
- Ask follow-up questions about a persisted document without uploading it again.
- Create Knowledge Spaces to group related documents.
- Ask questions across all documents in a Knowledge Space.
- Connect through Streamable HTTP or legacy SSE.
- Use structured MCP tool responses through `structuredContent`.
- Access reusable prompts through `prompts/list`.
- Access server documentation and tool catalogs through `resources/list`.
- Authenticate with the same Claix API key used for REST and A2A.

## Quick start

### 1. Create a Claix account and API key

Create a workspace at [claix.dev](https://claix.dev), then create an API key.

Your Claix API key authenticates requests to the MCP server:

```text
x-api-key: YOUR_CLAIX_API_KEY
```

> [!IMPORTANT]
> Never commit API keys to source control, paste them into public issues, include them in screenshots, or expose them in browser-side code. Use environment variables or your client’s secure secrets configuration.

### 2. Connect your MCP client

Use the hosted MCP endpoint:

```text
[https://claix.dev/mcp](https://claix.dev/mcp)
```

### 3. Discover schemas

Call:

```text
claix.schemas.list
```

This returns schemas in your Claix workspace, including:

- `schema_id`
- Schema name
- Document type
- Whether Agent Mode is enabled
- Agent definition metadata, when applicable

### 4. Extract a document

Use the extraction tool matching the input type:

```text
claix.extract.pdf
claix.extract.excel
claix.extract.doc
claix.extract.image
claix.extract.text
```

Provide the appropriate `schema_id` and document content.

### 5. Read structured output

Claix returns structured results through:

```text
structuredContent.data
```

A typical response contains:

```json
{
  "success": true,
  "data": {
    "invoice_number": "INV-2026-0841",
    "supplier_name": "Northwind Supplies",
    "total_amount": 1284.5,
    "currency": "EUR"
  }
}
```

## Install with Smithery

The fastest way to run the server through Smithery is:

```bash
npx -y @smithery/cli run info-f4xz/claix
```

You will be prompted to provide your Claix API key.

Manual Smithery configuration:

```json
{
  "mcpUrl": "[https://claix.dev/mcp](https://claix.dev/mcp)",
  "headers": {
    "x-api-key": "YOUR_CLAIX_API_KEY"
  }
}
```

## Connect with Cursor

Open:

```text
Cursor → Settings → MCP
```

Add this configuration:

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

Restart or reconnect Cursor if needed.

After connecting, your client can discover Claix tools through:

```text
tools/list
```

## Connect with Claude Desktop

Claude Desktop can connect through `mcp-remote` using the legacy SSE transport.

Add the following to your Claude Desktop configuration:

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

Replace:

```text
YOUR_CLAIX_API_KEY
```

with your real API key.

Restart Claude Desktop after saving the configuration.

## Connect with Claude Code

Use an MCP configuration that supports remote Streamable HTTP servers and set the Claix API key as a secure header:

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

The exact configuration location can differ by Claude Code version and operating system. Use the same endpoint and `x-api-key` header in the relevant MCP settings.

## Connect with Windsurf, Cline, Continue, Zed, Lovable, or Replit Agent

Use this server configuration wherever your MCP client accepts a remote MCP URL:

```json
{
  "url": "[https://claix.dev/mcp](https://claix.dev/mcp)",
  "headers": {
    "x-api-key": "YOUR_CLAIX_API_KEY"
  }
}
```

For clients that cannot send custom HTTP headers, supported Claix tools may accept an `api_key` argument as a fallback.

Header authentication remains the recommended option.

## Authentication

### Recommended: API key header

Send the Claix API key with every MCP request:

```http
x-api-key: YOUR_CLAIX_API_KEY
```

Example:

```http
POST /mcp HTTP/1.1
Host: claix.dev
Content-Type: application/json
Accept: application/json, text/event-stream
x-api-key: YOUR_CLAIX_API_KEY
```

### Fallback: per-tool API key

Some tools support an optional `api_key` argument for MCP clients that cannot send custom headers.

Example conceptual tool arguments:

```json
{
  "schema_id": "YOUR_SCHEMA_UUID",
  "file_base64": "JVBERi0xLjQK...",
  "api_key": "YOUR_CLAIX_API_KEY"
}
```

Prefer the `x-api-key` header whenever your client supports it.

## Transport modes

Claix supports two MCP transport modes on the same endpoint.

| Mode | Typical clients | Connection |
|---|---|---|
| Streamable HTTP | Cursor, Smithery, modern MCP clients, automation tools | `POST https://claix.dev/mcp` |
| Legacy SSE | Claude Desktop through `mcp-remote` and legacy session clients | `GET /mcp` plus `POST /mcp/message?sessionId=...` |

### Streamable HTTP

Use JSON-RPC 2.0 requests against:

```text
POST [https://claix.dev/mcp](https://claix.dev/mcp)
```

Recommended headers:

```http
Content-Type: application/json
Accept: application/json, text/event-stream
x-api-key: YOUR_CLAIX_API_KEY
```

### Legacy SSE

Use:

```text
GET [https://claix.dev/mcp](https://claix.dev/mcp)
```

to open the event stream. Then send messages to:

```text
POST [https://claix.dev/mcp/message?sessionId=YOUR_SESSION_ID](https://claix.dev/mcp/message?sessionId=YOUR_SESSION_ID)
```

The server automatically detects the transport mode.

## Initialize an MCP session

Example using Streamable HTTP:

```bash
curl -X POST "[https://claix.dev/mcp](https://claix.dev/mcp)" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "x-api-key: YOUR_CLAIX_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 0,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "claix-mcp-example",
        "version": "1.0.0"
      }
    }
  }'
```

## List available tools

```bash
curl -X POST "[https://claix.dev/mcp](https://claix.dev/mcp)" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "x-api-key: YOUR_CLAIX_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list",
    "params": {}
  }'
```

The response contains tool descriptions, input schemas, output schemas, and annotations. Destructive tools expose `destructiveHint` annotations.

## List Claix schemas

Before extracting a document, list schemas in the current Claix workspace:

```bash
curl -X POST "[https://claix.dev/mcp](https://claix.dev/mcp)" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "x-api-key: YOUR_CLAIX_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "claix.schemas.list",
      "arguments": {}
    }
  }'
```

Example response shape:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "structuredContent": {
      "success": true,
      "data": {
        "schemas": [
          {
            "id": "YOUR_SCHEMA_UUID",
            "name": "Invoice",
            "type": "pdf",
            "is_agent_mode": false
          }
        ]
      }
    }
  }
}
```

## Extract a PDF

Use `claix.extract.pdf` to convert a PDF into schema-validated JSON.

```bash
curl -X POST "[https://claix.dev/mcp](https://claix.dev/mcp)" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "x-api-key: YOUR_CLAIX_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "claix.extract.pdf",
      "arguments": {
        "schema_id": "YOUR_SCHEMA_UUID",
        "file_base64": "JVBERi0xLjQK..."
      }
    }
  }'
```

The `file_base64` value can be a Base64-encoded document or a data URL, depending on the tool input schema.

> [!NOTE]
> `claix.extract.pdf` supports text and scanned PDFs up to 15 MB.

## Extract other input types

| Input type | Tool | Input |
|---|---|---|
| Excel / CSV | `claix.extract.excel` | `file_base64` |
| PDF | `claix.extract.pdf` | `file_base64` |
| Document | `claix.extract.doc` | `file_base64` |
| Image | `claix.extract.image` | `file_base64` |
| Text / HTML / XML | `claix.extract.text` | `content` |

Supported document inputs include:

```text
Excel / CSV
PDF
DOCX
TXT
Markdown
RTF
JPEG
PNG
WebP
HEIC
HTML
XML
Plain text
```

Use an optional `space_id` when you want to add the extracted document to a Knowledge Space.

## Agent Mode

Agent Mode is intended for schemas configured with:

```text
is_agent_mode = true
```

Use the Agent Mode tools when your schema has this mode enabled:

| Input type | Tool |
|---|---|
| Excel / CSV | `claix.agent.excel` |
| PDF | `claix.agent.pdf` |
| Document | `claix.agent.doc` |
| Image | `claix.agent.image` |
| Text / HTML / XML | `claix.agent.text` |

Agent Mode returns the extracted result and additional agent-oriented analysis in `agent_data`.

Recommended flow:

```text
1. Call claix.schemas.list
2. Find a schema where is_agent_mode is true
3. Use the corresponding claix.agent.* tool
4. Read data[] and agent_data from structuredContent
```

## Schema management tools

| Tool | Description |
|---|---|
| `claix.schemas.list` | Lists schemas in the current account |
| `claix.schemas.create` | Creates a schema with a name, type, and schema definition |
| `claix.schemas.delete` | Deletes a schema by `schema_id` |

A schema defines the exact structured fields Claix should return.

Example

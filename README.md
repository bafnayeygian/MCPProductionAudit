# MCPProductionAudit

**MCPProductionAudit** is a local-first production-readiness, security, migration, and observability auditor for Model Context Protocol (MCP) traffic captures and tool catalogs.

Version: **1.0.0**

It is intentionally **not** a replacement for the official MCP conformance suite.

The official conformance project answers:

> Does this client/server implement protocol scenarios correctly?

MCPProductionAudit answers a different production question:

> Does captured real-world MCP traffic look secure, migration-ready, routable, cache-aware, schema-safe, observable, and compatible with the operational expectations of the target MCP revision?

## Why this project exists

MCP `2026-07-28` is a major protocol transition:

- stateless core
- no `initialize` / `initialized` lifecycle
- no protocol session header
- per-request metadata
- `Mcp-Method` / `Mcp-Name` HTTP routing headers
- cache hints for list/resource results
- Multi Round-Trip Requests
- full JSON Schema 2020-12 tool schemas
- authorization hardening
- deprecated roots, sampling, logging, and legacy HTTP+SSE
- documented W3C Trace Context propagation

Those changes create a practical need for migration and production-capture auditing.

## Main capabilities

### Protocol audit

- MCP protocol-version header
- JSON-RPC 2.0
- `Mcp-Method` presence
- `Mcp-Method` vs JSON body consistency
- `Mcp-Name` vs request name consistency
- stateless per-request `_meta`
- legacy initialization detection
- legacy session header detection

### Migration audit

- `initialize`
- `notifications/initialized`
- `Mcp-Session-Id`
- deprecated roots/sampling/logging families
- legacy HTTP+SSE
- legacy missing-resource `-32002`

### Cache audit

For:

- `tools/list`
- `prompts/list`
- `resources/list`
- `resources/read`

Checks:

- `ttlMs`
- `cacheScope`
- TTL type/range

### MRTR audit

Checks:

- `resultType: input_required`
- `inputRequests`
- `requestState`
- retry `inputResponses`
- echoed `requestState`

### Tool-schema audit

- duplicate tool names
- missing `inputSchema`
- `inputSchema` object-root rule
- external `$ref` warning
- configurable schema-depth limit
- output-schema traversal

MCPProductionAudit performs static production-safety checks. It does not claim to implement the entire JSON Schema 2020-12 validation vocabulary.

### Security audit

- credential-like query-string parameters
- optional expected Authorization header
- non-TLS remote MCP URLs
- token-passthrough metadata
- token-audience-validation metadata
- authorization issuer-validation metadata

Sensitive header values are never included in findings.

### Observability

- W3C `traceparent` shape
- trace baggage size

### Reports

- console
- JSON
- Markdown
- standalone HTML
- SARIF 2.1.0

SARIF output is suitable for GitHub Code Scanning ingestion.

### History and regression

- SQLite audit history
- baseline/candidate comparison
- score regression
- new-error regression
- security regression
- schema regression
- migration-blocker regression

## Install

```bash
git clone https://github.com/bafnayeygian/MCPProductionAudit.git
cd MCPProductionAudit
python -m pip install .
```

## Capture format

One JSON object per line:

```json
{
  "id": "tool-call-1",
  "transport": "streamable-http",
  "request": {
    "method": "POST",
    "url": "https://mcp.example.com/mcp",
    "headers": {
      "MCP-Protocol-Version": "2026-07-28",
      "Mcp-Method": "tools/call",
      "Mcp-Name": "search"
    },
    "body": {
      "jsonrpc": "2.0",
      "id": 1,
      "method": "tools/call",
      "params": {
        "name": "search",
        "arguments": {
          "q": "otters"
        },
        "_meta": {
          "io.modelcontextprotocol/protocolVersion": "2026-07-28",
          "io.modelcontextprotocol/clientInfo": {
            "name": "my-app",
            "version": "1.0"
          },
          "io.modelcontextprotocol/clientCapabilities": {}
        }
      }
    }
  },
  "response": {
    "status": 200,
    "headers": {
      "content-type": "application/json"
    },
    "body": {
      "jsonrpc": "2.0",
      "id": 1,
      "result": {
        "content": []
      }
    }
  }
}
```

## Audit

```bash
mcp-audit audit capture.jsonl
```

Target an older revision:

```bash
mcp-audit audit capture.jsonl --target-spec 2025-11-25
```

Use a policy:

```bash
mcp-audit audit capture.jsonl --policy policy.json
```

## CI gate

```bash
mcp-audit audit capture.jsonl \
  --min-score 90 \
  --fail-on-error \
  --sarif mcp-audit.sarif
```

## SQLite history

```bash
mcp-audit audit capture.jsonl \
  --db audits.db \
  --name baseline
```

Then:

```bash
mcp-audit runs --db audits.db
```

## Compare

```bash
mcp-audit compare \
  --db audits.db \
  --baseline 1 \
  --candidate 2 \
  --max-score-regression 5 \
  --max-new-errors 0
```

## Generate reports

```bash
mcp-audit report \
  --db audits.db \
  --run 1 \
  --html report.html \
  --markdown report.md \
  --json report.json \
  --sarif report.sarif
```

## GitHub Actions example

```yaml
- name: Audit captured MCP traffic
  run: |
    mcp-audit audit fixtures/mcp-capture.jsonl \
      --target-spec 2026-07-28 \
      --min-score 90 \
      --fail-on-error \
      --sarif mcp-audit.sarif
```

## Rules

Current rule families:

```text
MCP0xx  protocol and request metadata
MCP1xx  migration and deprecations
MCP2xx  cache behavior
MCP3xx  MRTR
MCP4xx  tool schemas
MCP5xx  security
MCP6xx  observability
MCP7xx  JSON-RPC compatibility
```

## Important scope

MCPProductionAudit audits recorded traffic and declarations.

It cannot prove runtime properties that are absent from the capture.

For example, OAuth issuer/audience checks are only reported when instrumentation metadata explicitly records their result.

The project avoids pretending that an unobservable security property was verified.

## References used for v1.0.0 design

- MCP 2026-07-28 release:
  https://blog.modelcontextprotocol.io/posts/2026-07-28/
- MCP official conformance project:
  https://github.com/modelcontextprotocol/conformance
- MCP TypeScript SDK 2026-07-28 migration documentation:
  https://ts.sdk.modelcontextprotocol.io/v2/migration/support-2026-07-28
- MCP authorization guidance:
  https://modelcontextprotocol.io/specification/draft/basic/authorization

## Roadmap

- HAR / mitmproxy capture import
- OpenTelemetry span import
- live passive proxy mode
- official conformance-result import
- JSON Schema 2020-12 optional validator integration
- OAuth metadata-document audit
- CIMD migration checks
- MCP Apps extension audit
- Tasks extension audit
- Enterprise Managed Authorization checks
- rule configuration and suppressions
- SARIF fingerprints
- JUnit output
- diff-by-rule HTML dashboard
- multi-service inventory
- fleet-level MCP governance reports

## License

MIT.

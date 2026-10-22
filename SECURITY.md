# Security policy

MCPProductionAudit is defensive analysis software.

## Capture sensitivity

MCP captures may contain:

- Authorization headers
- tool arguments
- tool results
- resource contents
- user data
- OAuth metadata

Do not commit sensitive captures.

## Secret handling

Findings never reproduce Authorization/Cookie/API-key header values.

Credential-like query parameters are reported by key name, not value.

## No active exploitation

v1.0.0 performs offline capture analysis and does not actively probe or exploit servers.

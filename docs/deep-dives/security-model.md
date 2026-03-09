# Security Model

Expose MCP servers only over secure channels. The server executes AI client commands against your database.

## Threat Model

1. **Unauthorized Access:** Discovery of the MCP server URL or auth token allows reading and writing to your knowledge graph.
2. **Interception:** Man-in-the-middle attacks on traffic between client and server.
3. **Credential Exposure:** Leakage of Supabase or API keys.

## Mitigations

### HTTPS

Serve the MCP server via HTTPS using a reverse proxy (e.g., Caddy) with Let's Encrypt certificates.

### Authentication

Require a strong bearer token (`openssl rand -hex 32`) for every request. Add this token to the `Authorization` header in TypingMind.

### Network Filtering

Restrict server access to known IP addresses via security groups or firewalls (UFW). This is practical only if your IPs are static; mobile users on dynamic IPs may need to skip this layer.

### Secrets Management

Store secrets in environment variables on your VPS. The MCP server reads from `process.env`, keeping secrets out of configuration files.

## The Connector

The [`@typingmind/mcp`](https://www.npmjs.com/package/@typingmind/mcp) package bridges TypingMind and your MCP servers. It handles HTTP/SSE transport and auth token injection.

## Limits

This model assumes control over both the TypingMind account and the VPS. If an attacker gains access to either endpoint, the architecture cannot provide further protection.

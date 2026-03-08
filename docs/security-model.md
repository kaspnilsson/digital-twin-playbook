# Security Model

Exposing an MCP server to the internet requires careful handling. The server accepts commands from your AI client and executes them against your database.

## The Threat Model

1. **Unauthorized access.** Someone discovers your MCP server URL and auth token. They can read your entire knowledge graph and write arbitrary data to it.
2. **Man-in-the-middle.** Traffic between your client and server is intercepted.
3. **Credential exposure.** Your Supabase keys or API keys leak.

## Mitigations

### HTTPS

The MCP server must be served over HTTPS. Use a reverse proxy like Caddy or Nginx with automatic certificate provisioning (Let's Encrypt). Never serve MCP over plain HTTP on the public internet.

### Auth Token

Generate a strong auth token (e.g., `openssl rand -hex 32`) and require it on every request:

```
Authorization: Bearer <your-token>
```

Include this header in your TypingMind MCP configuration. Treat this token like a password.

### Network-Level Restrictions

If you know your IP addresses, restrict access at the firewall level. Cloud providers offer security groups or UFW rules that whitelist specific IPs.

### Secrets Management

Do not commit secrets to git. Store them in environment variables on your VPS, or use a secrets manager. The MCP server reads from `process.env` -- it does not need secrets in config files.

## The MCP Connector

The [`@typingmind/mcp`](https://www.npmjs.com/package/@typingmind/mcp) package acts as a bridge between TypingMind and your MCP servers. It handles the HTTP/SSE transport and auth token injection. Run it behind Caddy with HTTPS.

## What This Does Not Protect Against

If an attacker gains access to your TypingMind account or your VPS, the architecture cannot help you. This model assumes you control both endpoints.

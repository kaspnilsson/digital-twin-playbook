# 01. Architecture Overview

Understand the Digital Twin architecture before provisioning servers. This distributed pipeline uses the Model Context Protocol (MCP) to connect your client to remote tools and memory.

## The Data Flow

```text
┌──────────────┐       ┌──────────────┐       ┌────────────────────────┐
│  TypingMind  │       │  Caddy Proxy │       │  MCP Gateway Node      │
│  (Client)    │ ────► │  (VPS: :443) │ ────► │  (@typingmind/mcp)     │
└──────────────┘ HTTPS └──────────────┘ HTTP  └──────┬───────┬───────┬─┘
       │                                             │       │       │ 
       │                                     stdio   │       │       │ stdio
       ▼                                     ┌───────▼─┐ ┌───▼───┐ ┌─▼─────┐
┌──────────────┐                             │ Memory  │ │ Brave │ │ GitHub│
│  OpenRouter  │ ◄────────────────────────── │ Server  │ │ Search│ │ Server│
│  (LLM API)   │                             │         │ │       │ │       │
└──────────────┘                             └───────┬─┘ └───┬───┘ └───┬───┘
                                                     │       │         │
                                              ┌──────▼─┐ ┌───▼───┐ ┌───▼───┐
                                              │Supabase│ │ Brave │ │GitHub │
                                              │(pgvect)│ │ API   │ │ API   │
                                              └────────┘ └───────┘ └───────┘
```

## Component Roles

### 1. The Client (TypingMind)
TypingMind is the interface. It runs as a Progressive Web App (PWA) on mobile and desktop. The client stores your LLM API keys and MCP credentials. It retrieves context from the MCP Gateway before routing prompts to the LLM.

### 2. The Reverse Proxy (Caddy)
Caddy secures connections to your VPS. Browsers block insecure `http://` requests from HTTPS applications like TypingMind. Caddy provisions SSL certificates via Let's Encrypt and forwards secure traffic to the internal Node process.

### 3. The MCP Gateway (`@typingmind/mcp`)
This Node.js background service runs on your VPS as a multiplexer. It authenticates HTTP/SSE connections via Bearer token, reads a `mcp-servers.json` registry, and spawns MCP tools via standard input/output (stdio).

### 4. The MCP Tools
The Gateway spawns four tools in this configuration:
- **Memory (`@kaspnilsson/mcp-memory-supabase`):** Manages knowledge graph persistence and semantic search via vector embeddings.
- **Web Search (`@brave/brave-search-mcp-server`):** Grounds responses in current events.
- **GitHub (`@modelcontextprotocol/server-github`):** Enables repository access, issue management, and PR reviews.
- **Sequential Thinking (`@modelcontextprotocol/server-sequential-thinking`):** Forces the model to plan multi-step operations before executing them.

You can add more MCP servers (local files, Slack, Google Drive) by updating the JSON registry.

### 5. The Database (Supabase)
Supabase stores entities, observations, and relations in managed Postgres.

## VPS vs. Local Setup

Cloud-hosting the MCP Gateway ensures device-agnostic context. While local servers bind capabilities to one machine, a cloud VPS allows your phone, laptop, and desktop to share the same database and search tools.

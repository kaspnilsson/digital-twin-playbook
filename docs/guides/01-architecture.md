# 01. Architecture Overview

Before provisioning servers, you need to understand how the components of the Digital Twin architecture communicate. 

This is not a monolithic application. It is a distributed pipeline built around the Model Context Protocol (MCP).

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

### 1. The Omni-Client (TypingMind)
The user interface. It runs as a Progressive Web App (PWA) on your phone, laptop, and desktop. It holds your LLM API keys (OpenRouter, OpenAI) and your MCP Gateway credentials. When you prompt the AI, the client queries the MCP Gateway for context (memory, search results) *before* routing the final prompt to the LLM.

### 2. The Reverse Proxy (Caddy)
TypingMind is a secure web application (HTTPS). Browsers strictly enforce mixed-content policies, meaning TypingMind cannot connect to an insecure `http://` endpoint. Caddy sits on your VPS, automatically provisions an SSL certificate via Let's Encrypt, and forwards secure traffic to the internal Node process.

### 3. The MCP Gateway (`@typingmind/mcp`)
This is the heart of the operation. It is a Node.js background service running on your VPS. It listens for incoming HTTP/SSE connections, authenticates them using a Bearer token, and acts as a **multiplexer**. 

It reads a `mcp-servers.json` configuration file, spawns multiple discrete MCP tools via standard input/output (stdio), and aggregates them into a single endpoint for the client.

### 4. The MCP Tools (The "Brain" and "Hands")
In this playbook, we configure the Gateway to spawn three essential tools:
- **Memory (`@kaspnilsson/mcp-memory-supabase`):** Reads and writes to a persistent Postgres database using vector embeddings for semantic search.
- **Web Search (`@brave/brave-search-mcp-server`):** Grounds the AI in current events. An AI without live search is a novelty; an AI with search is a utility.
- **GitHub (`@modelcontextprotocol/server-github`):** Allows the AI to read your repositories, open issues, and review PRs.

Because the Gateway relies on a simple JSON registry, this architecture is infinitely extensible. You can add MCP servers for your local filesystem, Slack, or Google Drive without changing the client.

### 5. The Database (Supabase)
A managed Postgres instance. It permanently stores the knowledge graph (entities, observations, and relations).

## Why this structure?

You might ask: *Why not just run the MCP servers locally on my laptop like Claude Desktop does?*

Because local servers bind your AI's capabilities to a single physical machine. By placing the MCP Gateway and its tools on a cloud VPS, your phone, your work laptop, and your personal desktop all read from and write to the exact same database, and all have the ability to search the web. Your context and your tools become device-agnostic.

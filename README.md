# Stop Letting AI Companies Monetize Your Memories

OpenAI remembers your conversations. So does Anthropic. So does Google. Each company stores what you tell their models, and each company makes that history available only inside their own product.

They are building a profile of how you think. What you work on. How you write. Who you talk about. And they are betting that once that profile is deep enough, you will never leave.

This should unsettle you.

I am a software engineer. Over the past year I have poured thousands of hours of my working context into AI tools -- project architectures, debugging sessions, design preferences, half-formed ideas. That context has genuine value. It makes every future conversation faster and sharper. And right now, it is scattered across four different vendors who would each prefer I forgot the other three exist.

So I built something different: a personal knowledge graph that sits in a Postgres database I own, accessible to any AI model I choose. Claude, GPT-4, Gemini, Llama -- they all read from and write to the same brain. When I switch models, I carry everything with me. When a company raises prices or degrades their product, I leave without losing a single memory.

## How it works

The [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) is an open standard that lets AI clients connect to external data sources. I wrote an MCP memory server called [`mcp-memory-supabase`](https://github.com/kaspnilsson/mcp-memory-supabase) that stores a knowledge graph -- entities, observations, and relations -- in Supabase with `pgvector` for semantic search.

I access it through [TypingMind](https://www.typingmind.com/), a BYO-key AI client that runs as a PWA on every device I own. My phone, my work laptop, my home desktop -- same interface, same memory, any model. I pick the best model for each task from a dropdown. The model inherits my full history instantly.

## What it costs

AI companies price subscriptions at $20/month to make the lock-in cheap enough that you do not question it. Here is what I actually pay:

- A VPS: ~$6/month (could be less; mine is oversized for other projects)
- Supabase: $0 (free tier)
- TypingMind: one-time license fee
- API compute via OpenRouter: ~$36 last month for 50 million tokens across 1,000+ requests

Yes, metered compute costs more than a subsidized subscription right now. The subscription price is artificially low. These companies are spending billions to acquire users before they monetize the data those users generate. I would rather pay the actual cost of inference and keep my data in a database I control.

## What does not work well

TypingMind is a Progressive Web App. On iOS, if you leave the app during a long tool-calling chain, the OS can kill the background process. Tough luck -- you have to re-run the query.

TypingMind syncs your chats across devices but does not yet sync MCP server configurations. You configure the server URL and auth token once per device. It takes two minutes, but it is a manual step.

And yes, this architecture replaces one set of dependencies with another. Instead of trusting OpenAI with your data, you trust Supabase and OpenRouter. The difference: every piece is replaceable. Swap Supabase for self-hosted Postgres. Swap OpenRouter for local Ollama. Swap TypingMind for any MCP-compatible client. Nothing about the architecture requires a specific vendor.

## Try it

1. Create a free [Supabase](https://supabase.com) project.
2. Run [`schema.sql`](https://github.com/kaspnilsson/mcp-memory-supabase/blob/main/supabase/schema.sql) in the SQL editor.
3. Add the server to your MCP client config:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@kaspnilsson/mcp-memory-supabase"],
      "env": {
        "SUPABASE_URL": "https://<your-project>.supabase.co",
        "SUPABASE_KEY": "<your-service-role-key>",
        "EMBEDDING_API_KEY": "<your-embedding-key>"
      }
    }
  }
}
```

The source is here: **[kaspnilsson/mcp-memory-supabase](https://github.com/kaspnilsson/mcp-memory-supabase)**

## Architecture Deep Dives

- [Why TypingMind?](docs/why-typingmind.md) -- The client requirements for this architecture.
- [Why Supabase?](docs/why-supabase.md) -- Managed Postgres vs. self-hosting.
- [Security Model](docs/security-model.md) -- How to expose an MCP server safely.
- [The Economics](docs/economics.md) -- A full cost breakdown vs. SaaS subscriptions.

---

I am working on a broader architecture guide -- a playbook for building this entire stack from scratch on a blank Linux box. If that interests you, star the repo and watch for updates.

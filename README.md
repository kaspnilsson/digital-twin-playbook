# Stop Letting AI Companies Monetize Your Memories

[OpenAI is exploring ads in ChatGPT](https://www.adweek.com/media/openai-chatgpt-ads-job-listing-marketing-platform/). [Anthropic's "Constitutional AI" guardrails are crumbling under pressure from the Pentagon](https://www.nbcnews.com/tech/tech-news/anthropic-says-pentagon-declared-national-security-risk-rcna262013). [Google trains on everything you feed Gemini](https://www.tomsguide.com/ai/google-gemini/how-to-stop-gemini-from-training-on-your-data). Nobody knows exactly what these companies will do with the thousands of conversations you have poured into their products -- your debugging sessions, your medical questions, your relationship problems, your business plans -- but the incentives are not subtle. They want you dependent on their product, and your conversation history is the lever.

Each of them remembers what you tell their models, and each of them locks that history inside their own app. Switch from Claude to ChatGPT and you start over. Switch to Gemini and you start over again. The profile they have built of how you think, how you write, and what you work on belongs to them, not to you. It does not follow you out the door.

When I'm not writing software to help develop the [Waymo Driver](https://waymo.com/), I'm an obsessive user of these tools. I've dumped a year of working context into them: project architectures, code reviews, personal health data, half-baked startup ideas. That accumulated context makes every conversation sharper. And it sits, today, in four separate vendor databases that do not talk to each other.

So I built my own system.

## What I Actually Built

I run a knowledge graph in a Postgres database on my own server. Claude, GPT-5, Gemini, GLM-5 -- every model I use reads from and writes to that same database. The data never touches a vendor's infrastructure. When I switch models, the new one picks up exactly where the old one left off.

The architecture has three layers:

### The Brain

I wrote an open-source MCP server called [`mcp-memory-supabase`](https://github.com/kaspnilsson/mcp-memory-supabase) that stores a knowledge graph -- entities, observations, relations -- in [Supabase](https://supabase.com/) (managed Postgres) with `pgvector` for semantic search. The [Model Context Protocol](https://modelcontextprotocol.io/) is an open standard that lets AI clients call external tools. This server is one of several tools I run.

### The Gateway

A small cloud VPS (virtual private server) hosts an MCP Gateway -- a Node.js daemon that multiplexes several MCP tools behind a single HTTPS endpoint. I run three: the memory server, [Brave Search](https://brave.com/search/api/) for grounding in live data, and a GitHub server for reading and writing code. The gateway authenticates requests with a bearer token and spawns each tool as a child process. Adding a new tool means adding four lines to a JSON config file.

### The Client

[TypingMind](https://www.typingmind.com/) connects to the gateway. It is a bring-your-own-key AI chat app that runs as a PWA on every device I own -- phone, work laptop, home desktop. Same interface, same tools, any model. I pick the model from a dropdown. A new release from Anthropic or OpenAI inherits my full history the moment I select it.

## What It Costs

AI subscriptions are priced at $20/month to make the lock-in feel cheap.

| Component | Monthly Cost |
|-----------|------|
| VPS | ~$6 |
| Supabase | $0 (free tier) |
| TypingMind | ~$3 amortized (one-time ~$40 license) |
| Brave Search API | $0 (free tier) |
| API compute via [OpenRouter](https://openrouter.ai/) | ~$36 for 50M tokens |
| **Total** | **~$45** |

Metered compute costs more than a flat subscription. But the $20 price is a customer-acquisition subsidy. These companies are burning cash to build a user base before they figure out how to extract value from the conversation data they are accumulating. I pay the actual cost of the compute and keep the data under my own lock and key.

## What Does Not Work Well

I will not pretend this competes with Claude.ai or ChatGPT as a *product*.

Anthropic has hundreds of engineers building a native application with voice, artifacts, projects, and a mobile app. TypingMind is a PWA built by a small team. The gaps show:

- **Voice is rough.** TypingMind supports speech input, but the experience is miles behind the native ChatGPT and Claude voice modes.
- **iOS kills background work.** If you leave the browser while the AI is mid-thought on a long tool chain, iOS may terminate the process. You have to re-run the query.
- **MCP config does not sync across devices.** TypingMind syncs chats, but you configure the MCP server URL and token manually on each device. Easy to work around, but annoying.
- **You are your own SRE.** If the VPS goes down at 2 AM, nobody pages anyone.

This is not a consumer product. If you want polish, use Claude.ai. If you want control, keep reading.

## Why It Gets Better Over Time

The subscription model gives you a fixed product. You get the features the vendor ships, on the vendor's timeline.

This system compounds. Every conversation enriches the same graph, regardless of which model I used or which device I was on. After a month of use, the AI knows my manager's name is James. It knows my height and weight and clothing sizes when I ask it to help me shop online. It knows I want options ranked in a rubric, not dumped in a vague pros-and-cons list. It knows the tech stack of every side project I maintain. I never re-explain any of this.

When Anthropic ships Sonnet 4.6, I select it from a dropdown. It reads the graph and immediately has the context that took me weeks to build. If I don't like how it behaves, I re-select Gemini 3.1 Pro and reissue the query. I do not wait for Anthropic or Google to build a "memory" feature and hope they implement it well. The memory is mine.

After three months of daily use, the distance between this setup and a fresh ChatGPT session is enormous -- and it grows with every conversation.

## The Playbook

This repository walks through the full architecture, start to finish. The guides are written for humans and structured for AI agents.

### Guides
1. [Architecture Overview](docs/guides/01-architecture.md) -- How the pieces fit together.
2. [Database Provisioning](docs/guides/02-database.md) -- Setting up Supabase and `pgvector`.
3. [VPS & Gateway Setup](docs/guides/03-vps-and-gateway.md) -- The server, Caddy, systemd, and the MCP multiplexer.
4. [The Client & System Prompt](docs/guides/04-client-setup.md) -- Connecting TypingMind and teaching the AI to use its tools.

### Deep Dives
- [The Economics](docs/deep-dives/economics.md) -- Full cost breakdown vs. SaaS subscriptions.
- [Security Model](docs/deep-dives/security-model.md) -- Exposing an MCP server safely.
- [Why TypingMind?](docs/deep-dives/why-typingmind.md) -- Client requirements for this architecture.
- [Why Supabase?](docs/deep-dives/why-supabase.md) -- Managed Postgres and the RLS debate.

### Open Source Primitives
- [`mcp-memory-supabase`](https://github.com/kaspnilsson/mcp-memory-supabase) -- The knowledge graph MCP server. MIT licensed.

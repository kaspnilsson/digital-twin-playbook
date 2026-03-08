# Stop Letting AI Companies Monetize Your Memories

OpenAI is exploring injecting ads into ChatGPT. Anthropic is contracting with the Department of Defense. Google trains on everything you feed Gemini. The details shift week to week, but the trajectory is clear: the companies storing your most intimate intellectual output -- your half-formed ideas, your debugging sessions, your medical questions, your relationship advice -- have interests that diverge sharply from yours.

Each company remembers your conversations, and each makes that history available only inside their product. They are building a profile of how you think. What you work on. How you write. Who you talk about. And they are betting that once that profile is deep enough, you will never leave.

This should unsettle you.

When I am not developing software for the [Waymo Driver](https://waymo.com/), I am an obsessive power user of AI tools. Over the past year I have poured thousands of hours of my working context into them -- project architectures, code reviews, design preferences, personal health data, half-baked startup ideas. That context has real value. It makes every future conversation faster and more precise. And right now, it is scattered across four different vendors who would each prefer I forgot the other three exist.

So I built something different.

## What I Actually Built

I run a personal knowledge graph in a Postgres database that I own. Every AI model I use -- Claude, GPT-4, Gemini, Llama, whatever ships next Tuesday -- reads from and writes to the same brain. The data lives on my own cloud server. When I switch models, I carry everything with me. When a company raises prices or ships ads, I leave without losing a thing.

The system runs on the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP), an open standard that lets AI clients connect to external tools. I wrote an open-source MCP memory server called [`mcp-memory-supabase`](https://github.com/kaspnilsson/mcp-memory-supabase) that stores the graph in Supabase with `pgvector` for semantic search. It is one component of a larger architecture:

- **A cloud VPS** hosts an MCP Gateway -- a Node process that multiplexes several tools (persistent memory, live web search via Brave, GitHub access) behind a single secure HTTPS endpoint.
- **[TypingMind](https://www.typingmind.com/)** acts as the client. It is a BYO-key AI chat app that runs as a PWA on every platform. My phone, my work laptop, my home desktop -- same interface, same memory, same tools, any model I want.
- **Supabase** (managed Postgres) stores the knowledge graph. Free tier. Automatic backups.

When a better model drops, I flip a dropdown in the TypingMind UI. The new model instantly inherits my entire memory graph, my custom tools, and my UI preferences. Zero migration cost.

## What It Costs

AI companies price subscriptions at $20/month to make the lock-in cheap enough that you stop thinking about it. Here is what I actually spend:

| Component | Cost |
|-----------|------|
| VPS (DigitalOcean) | ~$6/month |
| Supabase | $0 (free tier) |
| TypingMind | One-time license (~$40) |
| Brave Search API | $0 (free tier) |
| API compute (OpenRouter) | ~$36/month for 50M tokens |

Yes, metered compute costs more than a subsidized subscription right now. The $20/month price exists because these companies are spending billions to acquire users before they figure out how to monetize the data those users generate. I would rather pay the real cost of inference and own the database.

## What It Cannot Do

I am not going to pretend this is a better *product* than Claude.ai or ChatGPT.

Anthropic employs hundreds of engineers building a polished native application. TypingMind is a one-person shop building a PWA. The gaps are real:

- **Voice chat barely works.** TypingMind supports it, but the experience is nowhere near the native ChatGPT or Claude voice interfaces.
- **iOS kills background processes.** TypingMind is a Progressive Web App. If you leave the browser during a long tool-calling chain, iOS may terminate the process mid-flight. You re-run the query.
- **MCP config does not sync.** TypingMind syncs your chats across devices, but you must manually configure the MCP server connection on each client. Two minutes per device, but it is a manual step.
- **You are the sysadmin.** If the VPS goes down at 2 AM, nobody pages anyone. You fix it when you wake up.

This is not a consumer product. It is an architecture for people who care more about data ownership than about polish.

## Why It Gets Better Over Time

The subscription model gives you a static product. You get the features the vendor ships, on the vendor's schedule, with the vendor's defaults.

This system compounds. Every conversation I have with any model enriches the same knowledge graph. After a month of daily use, the AI knows that my manager's name is James without me explaining it. It knows my clothing sizes when I ask it to help me shop. It knows I prefer ranked comparisons with explicit rubrics over vague pros-and-cons lists. It knows the tech stack of every project I maintain.

A new Claude model inherits all of that context on the day it launches. I do not re-teach it anything. I do not wait for Anthropic to build a "memory" feature and hope they implement it the way I want. The memory is mine, and it transfers instantly.

The longer I use it, the wider the gap grows between this system and a fresh ChatGPT session.

## The Playbook

This repository is a start-to-finish guide for building the architecture I just described. It is written to be read by humans and executed by AI agents.

### Guides
1. [Architecture Overview](docs/guides/01-architecture.md) -- How the pieces fit together.
2. [Database Provisioning](docs/guides/02-database.md) -- Setting up Supabase and `pgvector`.
3. [VPS & Gateway Setup](docs/guides/03-vps-and-gateway.md) -- The server, Caddy, systemd, and the MCP multiplexer.
4. [The Client & System Prompt](docs/guides/04-client-setup.md) -- Connecting TypingMind and teaching the AI to use its tools.

### Deep Dives
- [The Economics](docs/deep-dives/economics.md) -- Full cost breakdown vs. SaaS subscriptions.
- [Security Model](docs/deep-dives/security-model.md) -- Exposing an MCP server to the internet safely.
- [Why TypingMind?](docs/deep-dives/why-typingmind.md) -- The client requirements for this architecture.
- [Why Supabase?](docs/deep-dives/why-supabase.md) -- Managed Postgres and the RLS debate.

### Open Source Primitives
- [`mcp-memory-supabase`](https://github.com/kaspnilsson/mcp-memory-supabase) -- The knowledge graph MCP server. MIT licensed.

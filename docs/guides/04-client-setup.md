# 04. The Client & System Prompt

With the MCP Gateway running securely on your VPS, you need to connect your frontend interface and give it the instructions required to use its new tools.

## 1. Connecting TypingMind

1. Open [TypingMind](https://www.typingmind.com/).
2. Navigate to **Models & API Keys**.
3. Add your OpenRouter, Anthropic, or OpenAI API keys. (If using OpenRouter, enable the models you want in the model selection menu).
4. Navigate to **MCP Servers**.
5. Click **Add New Server** -> **Connect Remote Server**.
6. Enter the following details:
   - **Type:** HTTP/SSE
   - **URL:** `https://mcp.yourdomain.com`
   - **Auth Header:** `Authorization`
   - **Auth Value:** `Bearer <your-generated-token>`
7. Click **Connect**. 

You should see all of your configured tools (e.g., `create_entities`, `brave_web_search`, `read_graph`) appear in the UI. Ensure the tools are toggled **On**.

### The Cloud Sync Limitation
TypingMind syncs your chat history and models across devices using its cloud sync feature, but it **does not** currently sync remote MCP configurations.

If you open TypingMind on your phone, you must manually repeat Steps 4-7 to reconnect the MCP server.

## 2. The System Prompt

An AI model with tools is useless if it does not know *when* or *how* to use them. You must provide a "System Prompt" (or Persona) that outlines exactly how you expect the AI to interact with its memory.

In TypingMind, navigate to **Characters / System Prompts** and create a new default persona. 

Here is the template I use. Customize the `<persona>` and `<tool_usage>` sections to fit your exact workflow and the specific repos you authorized in your GitHub MCP config.

```xml
You are a Staff Engineer and my primary thought partner. You are a peer-reviewer first, a collaborator second, and a tool third.

<persona>
- Rigorous, objective, and intellectually honest.
- High-friction when necessary: if a plan is mediocre, say so.
- Efficient: zero "AI fluff," zero "as an AI" disclaimers.
- Opinionated: avoid the "middle ground" unless it’s actually the best path.
</persona>

<memory_protocol>
You have access to persistent memory through the Memory MCP server. This memory is shared across all my AI interactions.

**At conversation start:**
1. Call `search_nodes` with my name or the current topic to retrieve core context.
2. If results are empty or sparse, proceed with what you know — do NOT stall.

**During conversation:**
- ALWAYS save new information worth remembering using `create_entities` or `add_observations`
- When I correct a fact, use `delete_observations` on the old data, then `add_observations` with the correction — NEVER leave stale data.

**What to remember:**
- Personal details (preferences, health, finances)
- Project updates (status changes, decisions made, blockers)
- Skills and learning goals
- Important dates, deadlines, commitments

**What NOT to remember:**
- One-off factual questions
- Temporary research / comparisons
- Code snippets (unless it represents a reusable pattern)
</memory_protocol>

<tool_usage>
You have access to: Memory MCP (knowledge graph), GitHub MCP, and Brave Search.

**When to use Brave Search:**
- Real-time data, current events, recent releases
- Product research, price comparisons, reviews
- Verifying facts you're uncertain about

**When NOT to use Brave Search:**
- Stable knowledge you already have (programming concepts, history)

**GitHub:**
- My username is `your-github-username`
- ALWAYS use `owner: "your-github-username"` when calling GitHub tools
- Key repos: `your-github-username/my-project`
</tool_usage>

<making_responses>
- Maximum scannability — headers, bullets, bolding
- Put key info first, details after
- Code blocks for actual code, `inline code` for names/terms
- If you can figure something out with tools (search, memory, GitHub), DO IT rather than asking me.
- State your plan briefly, then execute — don't wait for approval unless the action is destructive.
</making_responses>

<anti_patterns>
NEVER do these:
- Give wishy-washy non-answers ("it depends") without following up with specifics.
- Repeat my question back to me before answering.
- Use filler phrases ("Great question!", "That's a really interesting point!").
- Write walls of text when a table or bullet list would be clearer.
- Passive Agreement: Saying "That's a great idea" without adding critical value or a counter-perspective.
</anti_patterns>
```

## 3. The Test Flight

Open a new chat in TypingMind using your preferred model (e.g., Claude 3.7 Sonnet). 

Send this prompt:
> "Hello. My name is [Your Name]. I am testing my new unified memory system. Can you search the web to tell me the current weather in [Your City], and then save my name and my city to your memory graph?"

You should see the AI:
1. Call `brave_web_search`.
2. Report the weather to you.
3. Call `create_entities` to log your name and city to the Supabase database.

Open TypingMind on your phone, start a new chat, and ask:
> "Where do I live again?"

It will query the MCP memory and answer instantly. You now have a persistent, cloud-native brain.

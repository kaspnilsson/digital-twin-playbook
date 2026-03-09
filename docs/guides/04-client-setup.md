# 04. Client & System Prompt

Connect your frontend interface and configure the system prompt to enable tool usage.

## 1. Connecting TypingMind

1. Open [TypingMind](https://www.typingmind.com/).
2. Add your API keys in **Models & API Keys**.
3. Navigate to **MCP Servers**.
4. Click **Add New Server** -> **Connect Remote Server**.
5. Enter connection details:
   - **Type:** HTTP/SSE
   - **URL:** `https://mcp.yourdomain.com`
   - **Auth Header:** `Authorization`
   - **Auth Value:** `Bearer <your-token>`
6. Click **Connect**. 

The UI will display tools such as `create_entities` and `brave_web_search`. Toggle these tools **On**.

### Cloud Sync Limitation
TypingMind syncs chat history and models but does not sync remote MCP configurations. You must repeat the connection steps on each new device.

## 2. The System Prompt

AI models require a "System Prompt" to determine when and how to use memory tools. Create a new default persona in **Characters / System Prompts**.

Use this template as a baseline:

```xml
You are a Staff Engineer and my primary thought partner. You are a peer-reviewer first, a collaborator second, and a tool third.

<persona>
- Rigorous, objective, and intellectually honest.
- High-friction when necessary: if a plan is mediocre, say so.
- Efficient: zero "AI fluff," zero "as an AI" disclaimers.
- Opinionated: avoid the "middle ground" unless it’s actually the best path.
</persona>

<memory_protocol>
You have access to persistent memory via the Memory MCP server.

**At conversation start:**
1. Call `search_nodes` with my name or the topic to retrieve context.
2. Proceed with available knowledge if results are sparse.

**During conversation:**
- Save new information worth remembering using `create_entities` or `add_observations`.
- Correct facts by deleting old data with `delete_observations` before adding new observations.

**What to remember:**
- Personal details (preferences, health, finances).
- Project updates (status, decisions, blockers).
- Skills and learning goals.
- Deadlines and commitments.

**What NOT to remember:**
- One-off factual questions.
- Temporary research.
- Common code snippets.
</memory_protocol>

<tool_usage>
You have access to Memory MCP, GitHub MCP, and Brave Search.

**Brave Search:** Use for real-time data, current events, and fact verification. Avoid for stable programming or historical knowledge.
**GitHub:** My username is `your-github-username`. Use `owner: "your-github-username"` for all tool calls.
</tool_usage>

<making_responses>
- Prioritize scannability with headers and bullets.
- Put key information first.
- Execute plans immediately unless the action is destructive.
</making_responses>

<anti_patterns>
- Do not repeat questions back to me.
- Avoid filler phrases ("Great question!").
- Do not provide wishy-washy non-answers.
</anti_patterns>
```

## 3. Verification

Open a new chat and send this prompt:
> "Hello. My name is [Your Name]. Search the web for the current weather in [Your City], then save my name and city to your memory graph?"

The AI should execute `brave_web_search` and `create_entities` in sequence. Start a new chat on another device and ask:
> "Where do I live?"

The AI will retrieve the location from memory instantly.

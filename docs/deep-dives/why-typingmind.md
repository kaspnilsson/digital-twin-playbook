# Why TypingMind?

The architecture requires a client that meets specific criteria:

1. **Bring Your Own Keys (BYOK)** -- You provide API keys for OpenAI, Anthropic, OpenRouter, etc. The client does not resell compute.
2. **Remote MCP server support** -- The client must connect to MCP servers over HTTP/SSE, not just local processes.
3. **Cross-device sync** -- Your chats and preferences sync across desktop and mobile.
4. **PWA support** -- The client runs in a browser on any operating system.

TypingMind checks all four boxes. As of early 2026, few alternatives exist that support arbitrary remote MCP servers.

## Tradeoffs

- **No native mobile apps.** TypingMind is a PWA. On iOS, background processes can be killed. Long-running tool chains may fail if you switch apps.
- **Manual MCP config sync.** TypingMind syncs chats but not remote MCP server configurations. You must configure each device once by hand.
- **One-time license fee.** Not open source. You pay once for the client, then bring your own API keys.

## Alternatives

- **Claude Desktop** -- Supports local MCP servers. Does not support remote MCP over HTTP. Tied to Anthropic's models.
- **Cursor** -- Supports local MCP. Primary focus is code completion, not general-purpose chat.
- **Open WebUI** -- Open source, self-hostable. MCP support is evolving.

If another client emerges that meets the four criteria above, you can swap TypingMind without changing anything else in the architecture.

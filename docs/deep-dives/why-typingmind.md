# Why TypingMind?

The Digital Twin architecture requires a client with these features:

1. **Bring Your Own Keys (BYOK):** Use your own API keys for LLM providers.
2. **Remote MCP Support:** Connect to MCP servers via HTTP/SSE.
3. **Sync:** Synchronize chats across desktop and mobile.
4. **PWA:** Run in a browser on any operating system.

TypingMind fulfills these requirements. Few alternatives currently support arbitrary remote MCP servers.

## Tradeoffs

- **No Native Mobile Apps:** As a PWA on iOS, TypingMind may lose background processes. Long-running tool chains fail if you switch apps.
- **Manual Configuration:** TypingMind syncs chat history but not remote MCP server settings. Configure each device once manually.
- **License:** Requires a one-time fee and is not open source.

## Alternatives

- **Claude Desktop:** Supports local MCP but lacks remote HTTP support. Also, model lock-in.
- **Cursor:** Focuses on code completion rather than general chat.
- **Open WebUI:** Open source, but MCP support remains early.

If another client develops these four features, you can replace TypingMind without altering the architecture.

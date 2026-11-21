# Harness adapters

The skill itself is harness-neutral: `SKILL.md` and `references/` contain no host-specific syntax. Every harness needs the same three things:

1. **This skill** and **`ideation-hub-requirements-agent`**, installed where the harness discovers skills.
2. **The Ideation Hub MCP server**: Streamable HTTP at `https://api.ideationhub.me/api/mcp`, with headers `X-IH-Client-Id` and `X-IH-Client-Secret` taken from the environment variables `IH_CLIENT_ID` and `IH_CLIENT_SECRET`. Never paste the secret into a file you commit.
3. **An agent credential** created at https://www.ideationhub.me/settings/agents (Suggest is recommended). Add the `delivery:write` permission if you want handoffs to record the `design_ready` event.

| Harness | File |
| --- | --- |
| Cursor | `cursor.md` |
| Claude Code | `claude-code.md` |
| OpenAI Codex | `codex.md` |
| Grok Build (CLI) | `grok-build.md` |
| Grok Bot | `grok-bot.md` |
| Any other MCP + Agent Skills host | `generic.md` |

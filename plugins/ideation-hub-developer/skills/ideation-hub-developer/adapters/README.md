# Harness adapters

Install steps for Cursor, Claude Code, Claude Desktop, Grok Bot, and other hosts are in the repository at `docs/install/`. The notes below only name where each host looks for skills. `SKILL.md` and `references/` contain no host-specific syntax.

Every harness needs the same three things:

1. **This developer plugin**, the **requirements plugin** (`ideation-hub-requirements-agent`), and the **platform plugin** (`ideation-hub-platform`), installed as sibling directories. Cursor cannot declare that dependency. The deprecated alias `ideation-hub-system-design` ships inside this plugin.
2. **The Ideation Hub MCP server**: Streamable HTTP at `https://api.ideationhub.me/api/mcp`, with headers `X-IH-Client-Id` and `X-IH-Client-Secret` taken from the environment variables `IH_CLIENT_ID` and `IH_CLIENT_SECRET`. Never paste the secret into a file you commit.
3. **An agent credential** created at https://www.ideationhub.me/settings/agents (Suggest is recommended). Add the `delivery:write` permission if you want handoffs to record the `design_ready` event. Add `status:write` to record implementation and test-coverage assessments. That permission never edits a title, body, or trace link. The deprecated alias `ideation-hub-system-design` only points at this skill; install this folder for the instructions.

| Harness | File |
| --- | --- |
| Cursor | `cursor.md` |
| Claude Code | `claude-code.md` |
| OpenAI Codex | `codex.md` |
| Grok Build (CLI) | `grok-build.md` |
| Grok Bot | `grok-bot.md` |
| Any other MCP + Agent Skills host | `generic.md` |

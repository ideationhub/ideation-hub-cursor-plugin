# Claude Code

**Plugin:** the repository has a `.claude-plugin/plugin.json` and a root `.mcp.json`. Add the repository as a plugin source and enable `ideation-hub`. It loads the three skills in `skills/` and the root MCP server, so `/ideation-hub:ideation-hub-developer` resolves. `/ideation-hub:ideation-hub-system-design` is a deprecated alias. Schema, REST, and MCP docs are the `ideation-hub-platform` skill. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` in the environment Claude Code starts from.

**Manual:** copy the skill folders `skills/ideation-hub-platform`, `skills/ideation-hub-developer`, and `skills/ideation-hub-requirements-agent` so they stay siblings. The skill names are unchanged (the deprecated alias is `ideation-hub-system-design`). Add the server to the project `.mcp.json`:

```json
{
  "mcpServers": {
    "ideation-hub": {
      "type": "http",
      "url": "https://api.ideationhub.me/api/mcp",
      "headers": {
        "X-IH-Client-Id": "${IH_CLIENT_ID}",
        "X-IH-Client-Secret": "${IH_CLIENT_SECRET}"
      }
    }
  }
}
```

Docs: https://code.claude.com/docs/en/skills/, https://code.claude.com/docs/en/plugins/overview.

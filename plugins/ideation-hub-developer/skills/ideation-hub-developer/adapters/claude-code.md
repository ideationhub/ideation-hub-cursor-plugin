# Claude Code

**Plugin:** the repository has a `.claude-plugin/plugin.json` and a root `.mcp.json`. Add the repository as a plugin source and enable the `ideation-hub` compatibility bundle. `skills/` entries are symlinks into the developer and requirements plugins, so `/ideation-hub:ideation-hub-developer` still resolves. `/ideation-hub:ideation-hub-system-design` is a deprecated alias. Schema, REST, and MCP docs are in `plugins/ideation-hub-platform`. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` in the environment Claude Code starts from.

**Manual:** copy the three directories `plugins/ideation-hub-platform`, `plugins/ideation-hub-developer`, and `plugins/ideation-hub-requirements` so they stay siblings. The skill names inside them are unchanged (`ideation-hub-requirements-agent`, `ideation-hub-developer`, and the deprecated alias `ideation-hub-system-design`). Add the server to the project `.mcp.json`:

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

# Claude Code

**Plugin:** the repository has a `.claude-plugin/plugin.json` and a root `.mcp.json`. Add the repository as a plugin source and enable the `ideation-hub` plugin; the skill is then available as `/ideation-hub:ideation-hub-system-design`. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` in the environment Claude Code starts from.

**Manual:** copy both skill folders into `.claude/skills/` (project) or `~/.claude/skills/` (user), and add the server to the project `.mcp.json`:

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

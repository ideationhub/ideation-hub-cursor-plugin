# Claude Code

**Plugin:** the repository has a `.claude-plugin/plugin.json` and a root `.mcp.json`. Add the repository as a plugin source and enable the `ideation-hub` plugin; the skill is then available as `/ideation-hub:ideation-hub-developer`. `/ideation-hub:ideation-hub-system-design` still resolves and is a deprecated alias that points at the developer skill. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` in the environment Claude Code starts from.

**Manual:** copy `skills/ideation-hub-requirements-agent`, `skills/ideation-hub-developer`, and the deprecated alias `skills/ideation-hub-system-design` into `.claude/skills/` (project) or `~/.claude/skills/` (user), and add the server to the project `.mcp.json`:

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

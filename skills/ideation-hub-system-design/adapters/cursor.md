# Cursor

**Plugin (recommended):** install the Ideation Hub plugin from the Cursor Marketplace, or import `https://github.com/ideationhub/ideation-hub-cursor-plugin` (Dashboard → Settings → Plugins → Import from Repo). It brings both skills and the MCP server. Enter `IH_CLIENT_ID` and `IH_CLIENT_SECRET` in the setup card.

**Manual:** copy both skill folders into `.cursor/skills/` (project) or `~/.cursor/skills/` (user). Cursor also reads `.agents/skills/` and `.claude/skills/`. Add the server to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ideation-hub": {
      "url": "https://api.ideationhub.me/api/mcp",
      "headers": {
        "X-IH-Client-Id": "${env:IH_CLIENT_ID}",
        "X-IH-Client-Secret": "${env:IH_CLIENT_SECRET}"
      }
    }
  }
}
```

Invoke with `/ideation-hub-system-design`, or ask for a system design and let the agent pick the skill. Docs: https://cursor.com/docs/skills, https://cursor.com/docs/mcp.

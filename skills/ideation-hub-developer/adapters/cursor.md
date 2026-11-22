# Cursor

**Plugin (recommended):** install the Ideation Hub plugin from the Cursor Marketplace, or import `https://github.com/ideationhub/ideation-hub-cursor-plugin` (Dashboard → Settings → Plugins → Import from Repo). It brings both skills and the MCP server. Enter `IH_CLIENT_ID` and `IH_CLIENT_SECRET` in the setup card.

**Manual:** copy `skills/ideation-hub-requirements-agent`, `skills/ideation-hub-developer`, and the deprecated alias `skills/ideation-hub-system-design` into `.cursor/skills/` (project) or `~/.cursor/skills/` (user). Cursor also reads `.agents/skills/` and `.claude/skills/`. Add the server to `.cursor/mcp.json`:

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

Invoke with `/ideation-hub-developer`, or ask for a system design, an implementation, or an assessment and let the agent pick the skill. `/ideation-hub-system-design` still resolves; it is a deprecated alias that points at the developer skill. Docs: https://cursor.com/docs/skills, https://cursor.com/docs/mcp.

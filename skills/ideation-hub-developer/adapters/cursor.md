# Cursor

**Plugin (recommended):** install **Ideation Hub** (`ideation-hub`) from the Cursor Marketplace once it is listed. Until then, in Customize choose From GitHub Repository and import `https://github.com/ideationhub/ideation-hub-cursor-plugin`, or on a Teams or Enterprise plan have an admin use Dashboard → Plugins & MCPs → Team Marketplaces → Import from Repo. The plugin brings the MCP server and all three skills. Enter `IH_CLIENT_ID` and `IH_CLIENT_SECRET` when Cursor asks.

**Manual:** copy the skill folders `skills/ideation-hub-platform`, `skills/ideation-hub-developer`, and `skills/ideation-hub-requirements-agent` so they stay siblings. `skills/ideation-hub-system-design` is the deprecated alias; copy it only if an older setup still asks for that name. Add the server to `.cursor/mcp.json`:

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

Invoke with `/ideation-hub-developer`, or ask for a system design, an implementation, or an assessment and let the agent pick the skill. `/ideation-hub-system-design` still resolves; it is a deprecated alias of the developer skill. Docs: https://cursor.com/docs/plugins, https://cursor.com/docs/skills, https://cursor.com/docs/mcp.

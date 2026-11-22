# Cursor

**Plugin (recommended):** import `https://github.com/ideationhub/ideation-hub-cursor-plugin` (Dashboard → Settings → Plugins → Import from Repo) and install **Ideation Hub Platform**, **Ideation Hub Developer**, and **Ideation Hub Requirements**. The platform plugin brings the MCP server. Enter `IH_CLIENT_ID` and `IH_CLIENT_SECRET` on that plugin. The root **Ideation Hub** bundle is the compatibility install. Do not install it together with the three plugins.

**Manual:** copy `plugins/ideation-hub-platform`, `plugins/ideation-hub-developer`, and `plugins/ideation-hub-requirements` so those three directories stay siblings. The developer plugin's `skills/` folder holds `ideation-hub-developer` and the deprecated alias `ideation-hub-system-design`. The requirements plugin's `skills/` folder holds `ideation-hub-requirements-agent`. Add the server to `.cursor/mcp.json` if you are not using the platform plugin's `mcp.json`:

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

Invoke with `/ideation-hub-developer`, or ask for a system design, an implementation, or an assessment and let the agent pick the plugin. `/ideation-hub-system-design` still resolves; it is a deprecated alias of the developer plugin. Docs: https://cursor.com/docs/plugins, https://cursor.com/docs/skills, https://cursor.com/docs/mcp.

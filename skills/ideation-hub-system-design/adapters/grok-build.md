# Grok Build (CLI)

**Plugin:** `grok plugin install ideationhub/ideation-hub-cursor-plugin --trust`. Grok Build also reads Claude Code plugins and the root `.mcp.json`. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` first.

**Manual:** copy both skill folders into `.grok/skills/` (project) or `~/.grok/skills/` (user); `~/.agents/skills/` also works. Add the server:

```bash
grok mcp add --transport http ideation-hub https://api.ideationhub.me/api/mcp \
  --header 'X-IH-Client-Id: ${IH_CLIENT_ID}' \
  --header 'X-IH-Client-Secret: ${IH_CLIENT_SECRET}'
```

Check it with `grok mcp doctor ideation-hub`. Docs: https://docs.x.ai/build/features/skills-plugins-marketplaces, https://docs.x.ai/build/features/mcp-servers.

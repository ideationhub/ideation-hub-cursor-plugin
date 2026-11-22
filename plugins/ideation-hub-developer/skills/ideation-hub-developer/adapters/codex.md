# OpenAI Codex

**Plugins:** Codex reads `.agents/skills/` in every directory from where you start it up to the repository root, and `~/.agents/skills/` for your user. Keep the three plugin directories as siblings so links into the platform plugin resolve, and point Codex at the skill folders inside them:

```bash
git clone https://github.com/ideationhub/ideation-hub-cursor-plugin /tmp/ih-plugin
mkdir -p ~/.cursor/plugins
cp -R /tmp/ih-plugin/plugins/ideation-hub-platform \
      /tmp/ih-plugin/plugins/ideation-hub-developer \
      /tmp/ih-plugin/plugins/ideation-hub-requirements \
      ~/.cursor/plugins/
```

**MCP:** add to `~/.codex/config.toml`:

```toml
[mcp_servers.ideation-hub]
url = "https://api.ideationhub.me/api/mcp"
env_http_headers = { "X-IH-Client-Id" = "IH_CLIENT_ID", "X-IH-Client-Secret" = "IH_CLIENT_SECRET" }
```

`env_http_headers` maps each header to the *name* of an environment variable, so the secret stays out of the file. Mention the developer plugin with `$ideation-hub-developer`. `$ideation-hub-system-design` still resolves; it is a deprecated alias. Docs: https://learn.chatgpt.com/docs/build-skills.

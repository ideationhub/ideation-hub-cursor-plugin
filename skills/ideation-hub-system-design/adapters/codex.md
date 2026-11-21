# OpenAI Codex

**Skills:** Codex reads `.agents/skills/` in every directory from where you start it up to the repository root, and `~/.agents/skills/` for your user. Copy or symlink both skill folders there:

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/ideationhub/ideation-hub-cursor-plugin /tmp/ih-plugin
cp -R /tmp/ih-plugin/skills/ideation-hub-requirements-agent ~/.agents/skills/
cp -R /tmp/ih-plugin/skills/ideation-hub-system-design ~/.agents/skills/
```

**MCP:** add to `~/.codex/config.toml`:

```toml
[mcp_servers.ideation-hub]
url = "https://api.ideationhub.me/api/mcp"
env_http_headers = { "X-IH-Client-Id" = "IH_CLIENT_ID", "X-IH-Client-Secret" = "IH_CLIENT_SECRET" }
```

`env_http_headers` maps each header to the *name* of an environment variable, so the secret stays out of the file. Mention the skill with `$ideation-hub-system-design` or pick it from `/skills`. Docs: https://learn.chatgpt.com/docs/build-skills.

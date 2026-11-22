# Grok Bot

Install the Ideation Hub plugin from the plugin catalog, or save each skill with the bot's skill tool (slugs `ideation-hub-requirements-agent` and `ideation-hub-developer`; `ideation-hub-system-design` is a deprecated alias that points at the developer skill) and add the MCP server as a connector named `IdeationHub` with URL `https://api.ideationhub.me/api/mcp` and headers `X-IH-Client-Id` / `X-IH-Client-Secret`. The bot opens a secure dialog for the secret; never paste it into chat.

# Grok Bot

Install the Ideation Hub platform, developer, and requirements plugins from the catalog, or save the agent instructions with the bot's skill tool (slugs `ideation-hub-requirements-agent` and `ideation-hub-developer`; `ideation-hub-system-design` is a deprecated alias of the developer plugin). The schema, REST API, and MCP catalog are the platform plugin. Add the MCP server as a connector named `IdeationHub` with URL `https://api.ideationhub.me/api/mcp` and headers `X-IH-Client-Id` / `X-IH-Client-Secret`. The bot opens a secure dialog for the secret; never paste it into chat.

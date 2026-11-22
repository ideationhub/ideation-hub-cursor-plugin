# Any other host

- **Skills:** any loader that follows the Agent Skills specification (https://agentskills.io/specification) can load `ideation-hub-requirements-agent` and `ideation-hub-developer`. Also load `ideation-hub-system-design` if existing installs still invoke that name; it is a deprecated alias whose `SKILL.md` points at the developer skill. If the host has no skill loader, give the agent the contents of `skills/ideation-hub-developer/SKILL.md` as instructions and let it read the `references/` files on demand.
- **MCP:** Streamable HTTP JSON-RPC, `POST https://api.ideationhub.me/api/mcp`, protocol `2025-03-26` (or `2024-11-05`), no SSE. Send `X-IH-Client-Id` and `X-IH-Client-Secret` on every request, or use an OAuth `client_credentials` bearer token (see the requirements skill's `references/rest-api.md`).
- **No subagents?** Where the requirements skill asks for a fresh-reader check, do its editor pass instead.

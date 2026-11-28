# Install Ideation Hub in Claude Code and Claude Desktop

These steps were checked on 22 November 2026 against https://code.claude.com/docs/en/plugins/create-marketplace, https://code.claude.com/docs/en/plugins/marketplace-reference, and https://code.claude.com/docs/en/mcp.

One plugin, `ideation-hub`, serves Cursor, Claude Code, and Grok Build. Cursor reads `.cursor-plugin/plugin.json`. Claude Code reads `.claude-plugin/plugin.json`. Both load the same `skills/` folder and the same root MCP file, so nothing is copied a second time.

The server address in this repository is https://api.ideationhub.me/api/mcp. On 22 November 2026 a request to https://www.ideationhub.me/api/mcp also returned a successful response. The files in this repository use the api host.

Create a credential before you connect the server. Sign in to Ideation Hub at https://www.ideationhub.me. Open Settings, then Agent credentials, at https://www.ideationhub.me/settings/agents. Create a credential and copy the client identifier and the client secret. The secret is shown only once. Do not put the secret in a chat, a commit, or a file you share.

## Claude Code plugins

The marketplace file is `.claude-plugin/marketplace.json` at the root of this repository. The Claude Code documentation says to register a GitHub repository with this command, using the owner and the repository name.

`claude plugin marketplace add ideationhub/ideation-hub-cursor-plugin`

Install the plugin by its name, an at-sign, and the marketplace name. This repository has one plugin, `ideation-hub`, and the marketplace is also named `ideation-hub`.

`claude plugin install ideation-hub@ideation-hub`

It loads the three skills (`ideation-hub-platform`, `ideation-hub-requirements-agent`, and `ideation-hub-developer`, plus the deprecated alias `ideation-hub-system-design`) and the MCP server. If you installed the earlier separate `ideation-hub-platform`, `ideation-hub-requirements`, or `ideation-hub-developer` plugins, uninstall them before installing this one, or the same instructions load twice.

The documentation says the same marketplace can be registered inside a session with `/plugin marketplace add`, and a plugin can be installed from the `/plugin` panel. Those commands were read from the create-marketplace page.

The plugin's MCP file is the root `.mcp.json`. It sends the headers `X-IH-Client-Id` and `X-IH-Client-Secret`. Claude Code expands `${IH_CLIENT_ID}` and `${IH_CLIENT_SECRET}` from the environment. Export both variables in the environment that starts Claude Code, using the credential you created. Do not write the secret into the file.

## Claude Code without plugins

The Model Context Protocol page for Claude Code says a project server lives in `.mcp.json` at the project root, and that a remote server needs `"type": "http"` plus an address. A header may use `${NAME}` for an environment variable. An entry with an address and no type is skipped.

Copy `docs/mcp/claude-code.mcp.json` to `.mcp.json` in the project. That file is the same file as the root `mcp.json`. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` before you start Claude Code.

The marketplace reference says Claude Code also auto-loads plugins from `.claude/skills/`. Not verified here: whether a skill folder copied into `.claude/skills/` follows links that leave that folder. To keep the links working, copy the three skill folders so they remain siblings, and keep the folder names `ideation-hub-platform`, `ideation-hub-developer`, and `ideation-hub-requirements-agent`.

## Claude Desktop

Not verified. The Claude Desktop configuration pages read for this document describe local servers under Settings, then Developer, and remote servers that an administrator deploys. They do not show an end-user file format for a remote server with custom headers. Do not assume the Claude Code `.mcp.json` file is read by Claude Desktop. Use Claude Code for the verified remote setup above, or ask the desktop application's current help before you invent a configuration.

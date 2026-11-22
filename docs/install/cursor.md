# Install Ideation Hub in Cursor

These steps were checked against the Cursor documentation on 22 November 2026. The pages used are https://cursor.com/docs/plugins, https://cursor.com/docs/skills, https://cursor.com/docs/mcp, and https://cursor.com/docs/reference/plugins.

Ideation Hub is a workspace for product requirements. This repository holds three plugins that share one set of files. The platform plugin describes the records and the tools. The requirements plugin is the writing guide. The developer plugin is the design and implementation guide. Cursor has no field that makes one plugin install another, so install the platform plugin together with each of the others.

The server address in this repository is https://api.ideationhub.me/api/mcp. On 22 November 2026 a request to https://www.ideationhub.me/api/mcp also returned a successful response. The files in this repository use the api host.

Create a credential before you connect the server. Sign in to Ideation Hub at https://www.ideationhub.me. Open Settings, then Agent credentials, at https://www.ideationhub.me/settings/agents. Create a credential and copy the client identifier and the client secret. The secret is shown only once. Do not put the secret in a chat, a commit, or a file you share.

## Install the plugins

The Cursor skills documentation says skills from a GitHub repository are not imported on their own. Package them in a plugin and import the repository in Customize with From GitHub Repository. The repository needs a `.cursor-plugin/marketplace.json` file. This repository has that file.

Open Customize in Cursor.

Import this repository, https://github.com/ideationhub/ideation-hub-cursor-plugin, with From GitHub Repository.

Install Ideation Hub Platform, Ideation Hub Requirements, and Ideation Hub Developer from that marketplace. Leave the older combined Ideation Hub entry uninstalled if you install those three. Installing both loads the same instructions twice.

The Cursor plugins documentation also describes a team marketplace for Teams and Enterprise plans. A team admin opens Dashboard, then Plugins & MCPs, then Team Marketplaces, chooses Add Marketplace, and uses Import from Repo with the repository address. Members then install the plugins from Customize. That page says only admins can add a team marketplace on an Enterprise plan.

## Connect the server without a plugin

The Cursor Model Context Protocol documentation says you can configure a server in a file named `mcp.json`. A project file is `.cursor/mcp.json`. A file for every project is `~/.cursor/mcp.json`. A remote server uses an address and optional headers. In those files, an environment variable is written as `${env:NAME}`.

Copy `docs/mcp/cursor.mcp.json` from this repository into one of those locations. Set the environment variables `IH_CLIENT_ID` and `IH_CLIENT_SECRET` to the credential you created. The file never contains the secret itself.

The same documentation says you can turn a server on or off from Customize in the sidebar.

## Skills only

If you only want the instruction folders, copy these three directories so they sit next to each other. The names must stay the same, because each folder's instructions point at the sibling named `ideation-hub-platform`.

`plugins/ideation-hub-platform/skills/ideation-hub-platform`

`plugins/ideation-hub-developer/skills/ideation-hub-developer`

`plugins/ideation-hub-requirements/skills/ideation-hub-requirements-agent`

The deprecated name `ideation-hub-system-design` is the folder beside the developer instructions, `plugins/ideation-hub-developer/skills/ideation-hub-system-design`. Copy it too if an older setup still asks for that name.

The Cursor skills page says a repository's skills arrive by installing the plugin, not by copying folders into Cursor yourself. Copying is for a host that reads skill folders directly. That limit is from the Cursor skills documentation, not a guess.

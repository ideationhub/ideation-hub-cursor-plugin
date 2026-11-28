# Install Ideation Hub in another harness

Use this page when the harness is not Cursor, Claude Code, Claude Desktop, or Grok Bot. Only the parts marked as checked come from a vendor document. Everything else is the layout of this repository.

The server address in this repository is https://api.ideationhub.me/api/mcp. On 22 November 2026 a request to https://www.ideationhub.me/api/mcp also returned a successful response. The files in this repository use the api host.

Create a credential first. Sign in to Ideation Hub at https://www.ideationhub.me. Open Settings, then Agent credentials, at https://www.ideationhub.me/settings/agents. Create a credential and copy the client identifier and the client secret. The secret is shown only once. Replace the angle-bracket placeholders in the sample file. Do not commit the filled-in file.

## A host that only loads skill folders

The Agent Skills specification at https://agentskills.io/specification describes a folder with a `SKILL.md` file and optional `references` files. This was checked on 22 November 2026. This repository already follows that layout.

Copy these three folders so they sit next to each other, and do not rename them.

`skills/ideation-hub-platform`

`skills/ideation-hub-developer`

`skills/ideation-hub-requirements-agent`

The developer and requirements folders point at `../ideation-hub-platform/references/` when they are siblings. Copy `skills/ideation-hub-system-design` as well if something still asks for the old name. That folder only points at the developer instructions.

## A host with neither plugins nor skills

`docs/instructions/ideation-hub-platform.md`, `docs/instructions/ideation-hub-developer.md`, and `docs/instructions/ideation-hub-requirements.md` are single files generated from those skill folders. Give the host the file for the work you want. Do not edit the generated files. Run `node scripts/build-instructions.mjs` after a source change. `node scripts/validate-template.mjs` fails if a generated file is stale.

## A host that can call a remote Model Context Protocol server

`docs/mcp/generic.mcp.json` is a sample. It uses the key `mcpServers`, the type `http`, the server address, and two headers, `X-IH-Client-Id` and `X-IH-Client-Secret`. Claude Code's documented project file uses that shape. This repository's plugin uses it too, with environment-variable placeholders instead of angle brackets.

Not verified: that any other product accepts this sample unchanged. Read that product's own Model Context Protocol page and adjust the key names before you rely on it. The header names and the server address are the Ideation Hub contract. The surrounding keys may differ.

Cursor's copy is `docs/mcp/cursor.mcp.json`. It uses `${env:IH_CLIENT_ID}` because that is the form in the Cursor documentation. Claude Code's copy is `docs/mcp/claude-code.mcp.json`, which is the root `mcp.json`, and it uses `${IH_CLIENT_ID}` because that is the form in the Claude Code documentation.

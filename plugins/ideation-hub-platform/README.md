# Ideation Hub platform plugin

Canonical description of the Ideation Hub schema, REST API, and MCP tools, plus the MCP server config for this repository.

The reference text lives in one skill folder, `skills/ideation-hub-platform/`, so a host that only loads skills can copy that folder. Cursor and Claude Code load the same folder from this plugin. The developer and requirements plugins point here instead of copying the text.

Cursor and Claude Code have no field for plugin dependencies. Install this plugin together with any plugin that names it. The requirement is written in those plugins' manifests, READMEs, and `SKILL.md` files.

## What is in this plugin

```text
ideation-hub-platform/
├── .cursor-plugin/plugin.json
├── .claude-plugin/plugin.json
├── mcp.json                  same server file Claude Code loads
├── README.md                 credential setup
├── references -> skills/ideation-hub-platform/references
└── skills/ideation-hub-platform/
    ├── SKILL.md
    └── references/           schema.md, mcp.md, rest-api.md
```

Cursor and Claude Code both read this directory. The reference text lives only under `skills/ideation-hub-platform/references/`. `references` at the plugin root is a link to that folder, so older paths keep working. A generated single file for hosts with no plugin and no skill loader is `docs/instructions/ideation-hub-platform.md`.

The same MCP server config is also kept at the repository root (`mcp.json`, `.mcp.json`, `.cursor-plugin/mcp.json`, `.grok-plugin/mcp.json`) so installs that already point at those files keep connecting. This plugin's `mcp.json` is the copy that belongs with the platform surface.

## Credential setup

Relay these steps in plain words. Never ask for a secret in chat.

1. Open `https://www.ideationhub.me/settings/agents?setup=grokbot` (a `next` query param is kept through sign-in) and sign in as an **Admin** of the Product.
2. On **Agent credentials** (`/settings/agents`; also Product → Agents) choose **Create**.
3. Enter a **Client ID**: letters, digits, `.` `_` `-`, starting with a letter, no spaces (for example `GrokBot`; `Grok Bot` fails). Pattern `^[A-Za-z][A-Za-z0-9._-]{2,63}$`.
4. Pick the permission for this product:
   - **Read** — `scopes: ["read"]`, `editDirectlyProductIds: []`. The agent can look, not write.
   - **Suggest** — `scopes: ["read", "propose"]`, `editDirectlyProductIds: []`. Writes land in the issuer's Review queue until a human Accepts. `suggest` is the same permission as `propose`.
   - **Edit** — `scopes: ["read", "propose"]` and `editDirectlyProductIds` set to the products this agent may change directly, or an `edit` scope on those products. Writes apply to the living record unless the call passes `mode: "suggest"` or `"propose"`.
   - Add `delivery:write` so a handoff can record `design_ready`. Add `status:write` so an assessment can be recorded. `status:write` never edits a title, body, or trace link.
5. Copy the **Client ID** and **Client Secret** from the masked secret card right away. The secret is shown **only once**.
6. Paste them into the host's credentials dialog as `IH_CLIENT_ID` and `IH_CLIENT_SECRET`. A host opens that dialog itself.
7. Confirm with `initialize` on `/api/mcp` or `ih_list_products`, echoing nothing.

Lost secret: create a new credential or rotate it on the same screen, then repeat step 5. The Admin who created the credential is the issuer and Accepts or Rejects proposals.

**Secrets rule:** never paste, echo, log, or print the secret or an access token. Reference env vars only. If a secret leaks, tell the human to rotate or revoke on the same screen.

## Install

From this repository's marketplace, install **Ideation Hub Platform** (`ideation-hub-platform`). Installing the legacy **Ideation Hub** bundle (`ideation-hub` at the repository root) still loads the developer and requirements instructions and the root MCP config. Install that bundle **or** the three split plugins, not both. Installing both loads the same instructions twice.

Manual install: copy this directory next to `ideation-hub-developer` and `ideation-hub-requirements` so they stay siblings. The other plugins' relative links assume that layout.

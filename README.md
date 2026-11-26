<p align="center">
  <img src="assets/logo.svg" alt="Ideation Hub Logo" width="84" height="84" />
</p>

<h1 align="center">Ideation Hub — Cursor & Grok Bot Plugin</h1>

<p align="center">
  <strong>Explicit product intent and traceable requirements for AI-first engineering teams.</strong><br />
  Connect coding agents to <a href="https://www.ideationhub.me">Ideation Hub</a> to read, validate, trace, and propose Business Needs, User Needs, PRDs, Designs, Implementations, and Tests via MCP.
</p>

<p align="center">
  <a href="https://www.ideationhub.me">Website</a> ·
  <a href="https://www.ideationhub.me/settings/agents?setup=grokbot">Get Agent Credentials</a> ·
  <a href="https://www.ideationhub.me/terms">Terms of Service</a> ·
  <a href="https://www.ideationhub.me/privacy">Privacy Policy</a>
</p>

---

## Overview

**From first idea to justified release confidence.**  
Ideation Hub gives AI-assisted engineering teams a living system of record for product intent—connecting **Business Needs (`BN`)**, **User Profiles**, **User Needs (`UN`)**, **Product Requirements (`PRD`)**, **System / UX / Component Designs (`SD`, `UXD`, `CD`)**, **Implementations (`IMPL`)**, and **Verification / Validation / Unit Tests (`VT`, `VER`, `UT`)** in a single traceable graph.

This repository is a multi-plugin marketplace:

1. **Ideation Hub Platform** (`plugins/ideation-hub-platform`) — the one description of the schema, REST API, and Model Context Protocol tools, plus `mcp.json` for `https://api.ideationhub.me/api/mcp`. The same text is the skill folder `skills/ideation-hub-platform` for hosts that only load skills.
2. **Ideation Hub Requirements** (`plugins/ideation-hub-requirements`, skill name `ideation-hub-requirements-agent`, 1.7.0) — authoring workflow and writing rubrics. Requires the platform plugin.
3. **Ideation Hub Developer** (`plugins/ideation-hub-developer`, skill name `ideation-hub-developer`, 0.4.0 beta) — system design, implementation, and assessment. Before designing or implementing, it traces each Product Requirement up to its User Needs. Assessment of the current revision is unchanged (PRD-180, applying PRD-026, PRD-172, and PRD-179). Requires the platform plugin. `ideation-hub-system-design` remains a deprecated alias.
4. **Ideation Hub Mobile Store Distribution** (`plugins/ideation-hub-mobile-store-distribution`, skill name `ideation-hub-mobile-store-distribution`, 0.1.0 beta) — download size, startup, offline use, on-device scanning, consumable purchases, and store privacy disclosures for a mobile app on the App Store and Google Play. Standalone. It does not require the platform plugin, and the compatibility bundle does not load it.
5. **Ideation Hub** (repository root, plugin name `ideation-hub`, 1.2.0) — compatibility bundle so existing installs keep loading the same instructions and the root MCP config. Install this bundle **or** the platform, developer, and requirements plugins, not both. Install the mobile store-distribution plugin in addition when you need it.

Cursor's marketplace format has no plugin-dependency field. The developer and requirements plugins document that they require the platform plugin. The mobile store-distribution plugin does not. Agent skill folder names and `SKILL.md` files stay, because that is how hosts load instructions. `skills/` at the repository root is a set of symlinks to those folders so the old paths still resolve.

With this plugin installed, Cursor and Grok Bot agents can:
- **Read & Search Intent (`Read`):** Fetch the full requirement hierarchy (`ih_get_chain`), search existing requirements (`ih_search_requirements`), inspect upstream/downstream trace neighbors (`ih_get_neighbors`), and validate candidate changes against the trace rubric (`ih_validate_against_trace`).
- **Propose Changes for Peer Review (`Suggest` — Default):** Queue structured requirement, profile, design, implementation, and test proposals (`ih_propose_requirement`, `ih_propose_edit`, `ih_propose_bulk_push`) into the human Review Queue on `www.ideationhub.me`—so agents never silently overwrite living specs.
- **Direct Authoring (`Edit` — Scoped):** Apply authorized updates and trace links (`ih_link_requirements`, `ih_create_requirement`, `ih_update_requirement`) with full agent + human issuer audit attribution.

---

## Quick Start & Setup

### 1. Mint Scoped Agent Credentials in Ideation Hub
1. Open **[`https://www.ideationhub.me/settings/agents?setup=grokbot`](https://www.ideationhub.me/settings/agents?setup=grokbot)** and sign in as an **Admin** of your Product.
2. Click **Create** under **Agent Credentials** (`/settings/agents` or **Product → Agents**).
3. Enter a **Client ID** (e.g., `CursorAgent` or `GrokBot` — letters, digits, `.`, `_`, `-`, starting with a letter, 3–64 chars).
4. Choose the permission scope for the target Product(s):
   - **Read:** Read and search requirements, profiles, and trace links only.
   - **Suggest (Recommended Default):** Read + queue proposals into your human Review Queue (`/api/me/requirements-review`) for acceptance.
   - **Edit:** Read + write directly to living records on explicitly allowed products (`editDirectlyProductIds`), with full audit trail attribution.
5. Copy the **Client ID** (`IH_CLIENT_ID`) and **Client Secret** (`IH_CLIENT_SECRET`) immediately (the secret is shown only once).

### 2. Configure the Plugin in Cursor or Grok Bot
When prompted by Cursor or Grok Bot's **Connect Apps** setup card, supply:

| Variable | Required | Secret | Description |
| --- | :---: | :---: | --- |
| `IH_CLIENT_ID` | Yes | No | Your Ideation Hub Agent Client ID (minted at `/settings/agents?setup=grokbot`) |
| `IH_CLIENT_SECRET` | Yes | Yes | Your Ideation Hub Agent Client Secret (shown once at creation) |

The plugin connects automatically to `https://api.ideationhub.me/api/mcp` using the `X-IH-Client-Id` and `X-IH-Client-Secret` request headers:

```json
{
  "mcpServers": {
    "ideation-hub": {
      "type": "http",
      "url": "https://api.ideationhub.me/api/mcp",
      "headers": {
        "X-IH-Client-Id": "${IH_CLIENT_ID}",
        "X-IH-Client-Secret": "${IH_CLIENT_SECRET}"
      }
    }
  }
}
```

### 3. Team Marketplace Instant Install (Optional)
Cursor Teams / Enterprise admins can import this repository directly with zero wait time via **Cursor Dashboard → Settings → Plugins → Import from Repo** using:
```text
https://github.com/ideationhub/ideation-hub-cursor-plugin
```

### 4. Other Agent Harnesses (Claude Code, OpenAI Codex, Grok Build)

The developer and requirements plugins keep `SKILL.md` files so any host that follows the open [Agent Skills specification](https://agentskills.io/specification) can load them. Install the platform plugin beside them.

- **Claude Code:** this repository includes `.claude-plugin/plugin.json` and a root `.mcp.json`. Enable the compatibility bundle, or copy the three directories under `plugins/` so they stay siblings. `skills/` at the repo root is symlinks to those plugins.
- **OpenAI Codex:** copy the plugin directories so they stay siblings (see the block below), and add the MCP server to `~/.codex/config.toml`:

  ```bash
  git clone https://github.com/ideationhub/ideation-hub-cursor-plugin /tmp/ih-plugin
  mkdir -p ~/.agents/skills
  # Install the plugins as siblings so relative links among the platform, developer, and requirements plugins resolve.
  # The mobile store-distribution plugin has no such links. Copy it too when you want that skill.
  mkdir -p ~/.cursor/plugins
  cp -R /tmp/ih-plugin/plugins/ideation-hub-platform \
        /tmp/ih-plugin/plugins/ideation-hub-developer \
        /tmp/ih-plugin/plugins/ideation-hub-requirements \
        /tmp/ih-plugin/plugins/ideation-hub-mobile-store-distribution \
        ~/.cursor/plugins/
  # Hosts that load agent skills from ~/.agents/skills can use the same folders' skills/.
  # Keep the plugin directories as siblings of each other. Do not flatten only the skill folders
  # or the links into the platform plugin break.
  # skills/ideation-hub-system-design inside the developer plugin is the deprecated alias.
  ```

  ```toml
  [mcp_servers.ideation-hub]
  url = "https://api.ideationhub.me/api/mcp"
  env_http_headers = { "X-IH-Client-Id" = "IH_CLIENT_ID", "X-IH-Client-Secret" = "IH_CLIENT_SECRET" }
  ```
- **Grok Build:** `grok plugin install ideationhub/ideation-hub-cursor-plugin --trust`.

Install guides, checked against the vendor pages named in each file: [Cursor](docs/install/cursor.md), [Claude Code and Claude Desktop](docs/install/claude.md), [Grok Bot](docs/install/grok-bot.md), and [other harnesses](docs/install/generic.md).

Ready-to-copy server files, with placeholders and no secrets, are in [docs/mcp](docs/mcp). One generated instruction file per unit is in [docs/instructions](docs/instructions).

Per-harness notes that ship inside the developer skill: [`plugins/ideation-hub-developer/skills/ideation-hub-developer/adapters/`](plugins/ideation-hub-developer/skills/ideation-hub-developer/adapters/).

---

## MCP tools

The tool catalog, permissions, and REST routes are the platform plugin: [`plugins/ideation-hub-platform/references/mcp.md`](plugins/ideation-hub-platform/references/mcp.md) and [`plugins/ideation-hub-platform/references/rest-api.md`](plugins/ideation-hub-platform/references/rest-api.md). With a **Suggest** credential, write tools file a proposal for human review. With **Edit**, the same tools change living records unless the call passes `mode: "suggest"`.

---

## Validation

To validate the plugin manifest, marketplace manifest, skill frontmatter, and referenced asset/MCP paths locally:

```bash
node scripts/validate-template.mjs
```

---

## Data Privacy, Security & Legal

- **No AI/ML Model Training:** Ideation Hub **does not** use your workspace data, requirements, proposals, code references, credentials, or MCP plugin requests/responses to train, fine-tune, or improve any AI or machine learning models.
- **Scoped & Revocable Access:** Agent credentials are hashed at rest, scoped to specific products and permissions (`Read`, `Suggest`, or `Edit`), and can be rotated or revoked at any time from [`https://www.ideationhub.me/settings/agents`](https://www.ideationhub.me/settings/agents).
- **Terms of Service:** [`https://www.ideationhub.me/terms`](https://www.ideationhub.me/terms)
- **Privacy Policy:** [`https://www.ideationhub.me/privacy`](https://www.ideationhub.me/privacy)
- **Support & Security Contact:** [`support@ideationhub.me`](mailto:support@ideationhub.me)
- **License:** [MIT](./LICENSE)

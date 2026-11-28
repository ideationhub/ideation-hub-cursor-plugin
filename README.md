<p align="center">
  <img src="assets/logo.svg" alt="Ideation Hub Logo" width="84" height="84" />
</p>

<h1 align="center">Ideation Hub — Cursor & Grok Bot Plugin</h1>

<p align="center">
  <strong>Explicit product intent and traceable requirements for AI-first engineering teams.</strong><br />
  Connect coding agents to <a href="https://www.ideationhub.me">Ideation Hub</a> to read, validate, trace, and propose Principles, Business Needs, User Needs, PRDs, Designs, Implementations, and Tests via MCP.
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
Ideation Hub gives AI-assisted engineering teams a living system of record for product intent—connecting **Principles (`PRN`)**, **Business Needs (`BN`)**, **User Profiles**, **User Needs (`UN`)**, **Product Requirements (`PRD`)**, **System / UX / Component Designs (`SD`, `UXD`, `CD`)**, **Implementations (`IMPL`)**, and **Verification / Validation / Unit Tests (`VT`, `VER`, `UT`)** in a single traceable graph.

This repository is one plugin, **Ideation Hub** (`ideation-hub`, 2.0.4). It ships the MCP server and three skills, side by side in `skills/`:

1. **`ideation-hub-platform`** (1.1.0) — the one description of the schema, REST API, and Model Context Protocol tools, plus how to create an agent credential. Both other skills read it.
2. **`ideation-hub-requirements-agent`** (1.7.2) — authoring workflow and writing rubrics for Business Needs, User Needs, and Product Requirements.
3. **`ideation-hub-developer`** (0.5.3 beta) — system design, implementation, and assessment. Before designing or implementing, it traces each Product Requirement up to its User Needs. Assessment of the current revision is unchanged (PRD-180, applying PRD-026, PRD-172, and PRD-179). `ideation-hub-system-design` is a deprecated alias that lives in `skills/ideation-hub-system-design`.

The MCP server is `https://api.ideationhub.me/api/mcp`. Each skill folder has one source of truth in `skills/`; there are no second copies. The skills read each other by relative path, so they stay siblings. Version 2.0.0 replaced the earlier four-entry marketplace (separate platform, requirements, and developer plugins plus a compatibility bundle) with this single plugin. Skill names did not change.

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

### 3. Install the plugin

**Default path: one plugin, one install.** Install **Ideation Hub** (`ideation-hub`) from the marketplace of your host. It brings the MCP server and all three skills.

- **Cursor:** once listed, install **Ideation Hub** from the Cursor Marketplace. Until then, in Customize choose **From GitHub Repository** and import `https://github.com/ideationhub/ideation-hub-cursor-plugin`, then install **Ideation Hub**. On Teams or Enterprise, an admin can instead use **Dashboard → Plugins & MCPs → Team Marketplaces → Import from Repo** with the same URL.
- **Grok Bot:** once the plugin is listed in Connect Apps, add **Ideation Hub** there. Until then, add a custom connector with the server URL and the two headers above, and save the skills' instructions with the bot's skill tool. See [docs/install/grok-bot.md](docs/install/grok-bot.md).
- **Claude Code:** `claude plugin marketplace add ideationhub/ideation-hub-cursor-plugin`, then `claude plugin install ideation-hub@ideation-hub`. Export `IH_CLIENT_ID` and `IH_CLIENT_SECRET` first.
- **Grok Build:** `grok plugin install ideationhub/ideation-hub-cursor-plugin --trust`.

### 4. Hosts without plugin support (OpenAI Codex, generic)

The three skill folders follow the open [Agent Skills specification](https://agentskills.io/specification), so any host that loads skills can use them. Copy them as siblings so the relative links resolve:

```bash
git clone https://github.com/ideationhub/ideation-hub-cursor-plugin /tmp/ih-plugin
mkdir -p ~/.agents/skills
cp -R /tmp/ih-plugin/skills/ideation-hub-platform \
      /tmp/ih-plugin/skills/ideation-hub-developer \
      /tmp/ih-plugin/skills/ideation-hub-requirements-agent \
      ~/.agents/skills/
# skills/ideation-hub-system-design is the deprecated alias of the developer skill; copy it only if something still asks for that name.
```

For OpenAI Codex also add the MCP server to `~/.codex/config.toml`:

```toml
[mcp_servers.ideation-hub]
url = "https://api.ideationhub.me/api/mcp"
env_http_headers = { "X-IH-Client-Id" = "IH_CLIENT_ID", "X-IH-Client-Secret" = "IH_CLIENT_SECRET" }
```

Install guides, checked against the vendor pages named in each file: [Cursor](docs/install/cursor.md), [Claude Code and Claude Desktop](docs/install/claude.md), [Grok Bot](docs/install/grok-bot.md), and [other harnesses](docs/install/generic.md).

Ready-to-copy server files, with placeholders and no secrets, are in [docs/mcp](docs/mcp). One generated instruction file per unit is in [docs/instructions](docs/instructions).

Per-harness notes that ship inside the developer skill: [`skills/ideation-hub-developer/adapters/`](skills/ideation-hub-developer/adapters/).

---

## MCP tools

The tool catalog, permissions, and REST routes are the platform skill: [`skills/ideation-hub-platform/references/mcp.md`](skills/ideation-hub-platform/references/mcp.md) and [`skills/ideation-hub-platform/references/rest-api.md`](skills/ideation-hub-platform/references/rest-api.md). With a **Suggest** credential, write tools file a proposal for human review. With **Edit**, the same tools change living records unless the call passes `mode: "suggest"`.

---

## Validation

To validate the plugin manifest, the marketplace manifest (exactly one plugin), skill frontmatter, and referenced asset/MCP paths locally:

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

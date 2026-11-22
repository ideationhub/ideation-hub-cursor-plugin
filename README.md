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

This plugin bundles:
1. **The Ideation Hub Hosted MCP Server (`https://api.ideationhub.me/api/mcp`)** — 27 typed Model Context Protocol tools over stateless Streamable HTTP JSON-RPC (`2025-03-26`).
2. **The `ideation-hub-requirements-agent` Skill (`skills/ideation-hub-requirements-agent/SKILL.md`)** — Design Controls workflow, layered Profile → UN → PRD writing rubrics, duplicate/orphan prevention, and propose-first governance rules.
3. **The `ideation-hub-developer` Skill (`skills/ideation-hub-developer/SKILL.md`, 0.2.0 beta)** — system design, implementation, and assessment of implementation and test coverage status for Ideation Hub Product Requirements. Collaborative, proportionate System, Component and UX Design records traced to Product Requirements, trade-off analysis, design validation, implementation, and assessment of the current revision (PRD-180, applying PRD-026, PRD-172, and PRD-179). Design changes are always filed as proposals unless you explicitly ask for a direct write. `skills/ideation-hub-system-design/` is a deprecated alias that points at this skill so existing installs still resolve the old name.

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

Both skills follow the open [Agent Skills specification](https://agentskills.io/specification), so they load in any compatible agent.

- **Claude Code:** this repository includes `.claude-plugin/plugin.json` and a root `.mcp.json`. Enable it as a plugin, or copy `skills/*` into `.claude/skills/`.
- **OpenAI Codex:** copy (or symlink) both skill folders into `.agents/skills/` in your repository, or `~/.agents/skills/` for every project, and add the MCP server to `~/.codex/config.toml`:

  ```bash
  git clone https://github.com/ideationhub/ideation-hub-cursor-plugin /tmp/ih-plugin
  mkdir -p ~/.agents/skills
  cp -R /tmp/ih-plugin/skills/ideation-hub-requirements-agent /tmp/ih-plugin/skills/ideation-hub-developer ~/.agents/skills/
  # Deprecated alias so hosts that still invoke ideation-hub-system-design resolve:
  cp -R /tmp/ih-plugin/skills/ideation-hub-system-design ~/.agents/skills/
  ```

  ```toml
  [mcp_servers.ideation-hub]
  url = "https://api.ideationhub.me/api/mcp"
  env_http_headers = { "X-IH-Client-Id" = "IH_CLIENT_ID", "X-IH-Client-Secret" = "IH_CLIENT_SECRET" }
  ```
- **Grok Build:** `grok plugin install ideationhub/ideation-hub-cursor-plugin --trust`.

Per-harness details: [`skills/ideation-hub-developer/adapters/`](skills/ideation-hub-developer/adapters/).

---

## Included MCP Tools (27 Tools)

With a **Suggest** credential every write tool files a proposal for human review. With **Edit** (direct edit on that product) the same tools change living records unless the call passes `mode: "suggest"`.

| Tool | Permission | Description |
| --- | --- | --- |
| `ih_list_products` | `read` | List products and folders this credential can open, with `directEdit` per product. |
| `ih_get_chain` | `read` | Profiles, requirements and trace links for a product (`includePending`, `includeObsolete`, tag filters). |
| `ih_get_requirement` | `read` | One requirement by human ID or UUID: revisions, design fields, tags, links and its pending proposal. |
| `ih_get_neighbors` | `read` | Walk trace neighbors up, down or both, to a given depth. |
| `ih_search` | `read` | Hybrid keyword and meaning search across requirements, profiles, sections, specs and pending proposals. |
| `ih_search_requirements` | `read` | Keyword and filter listing by human ID, title and statement. |
| `ih_list_review_queue` | `read` | Proposals visible to this credential, with what each change does. |
| `ih_validate_against_trace` | `read` | Check a record or draft for duplicate titles, missing profile or parent, and orphans. Writes nothing. |
| `ih_list_sections` | `read` | PRD sections in order, with summaries. |
| `ih_propose_requirement` | `suggest` / `edit` | Add a BN, UN, PRD, SD, UXD, CD, IMPL, VT, VER or UT record. |
| `ih_propose_edit` | `suggest` / `edit` | Revise a requirement, including link changes; amends the one pending proposal. |
| `ih_propose_bulk_push` | `suggest` / `edit` | Validate and apply or queue many records and links together (`dryRun` supported). |
| `ih_propose_profile` | `suggest` / `edit` | Add or edit a User Profile. |
| `ih_link_requirements` | `suggest` / `edit` | Add a trace link (`addresses`, `realized_by`, `refined_by`, `checked_by`). |
| `ih_unlink_requirements` | `suggest` / `edit` | Soft-remove a trace link. |
| `ih_lifecycle_action` | `suggest` / `edit` | Withdraw, reopen, obsolete, delete draft or reclassify. |
| `ih_create_section` | `suggest` / `edit` | Add a PRD section. |
| `ih_rename_section` | `suggest` / `edit` | Rename or nest a PRD section. |
| `ih_delete_section` | `suggest` / `edit` | Delete a PRD section (requirements become unassigned). |
| `ih_reorder_sections` | `suggest` / `edit` | Reorder all PRD sections. |
| `ih_set_section_summary` | `suggest` / `edit` | Set a PRD section summary. |
| `ih_assign_sections` | `suggest` / `edit` | Place many PRDs into sections in one change. |
| `ih_create_requirement` | `edit` only | Direct-only add. |
| `ih_update_requirement` | `edit` only | Direct-only revise. |
| `ih_set_tags` | `edit` only | Replace a requirement's tags (no new revision). |
| `ih_restore_draft_title` | `suggest` (own pending add) | Title-only repair of a pending add. |
| `ih_record_development_phase_event` | `delivery:write` (direct) | Append one development-phase event (for example `design_ready`). Not a proposal. |

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

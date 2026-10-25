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
1. **The Ideation Hub Hosted MCP Server (`https://api.ideationhub.me/api/mcp`)** — 16 typed Model Context Protocol tools over stateless Streamable HTTP JSON-RPC (`2025-03-26`).
2. **The `ideation-hub-requirements-agent` Skill (`skills/ideation-hub-requirements-agent/SKILL.md`)** — Design Controls workflow, layered Profile → UN → PRD writing rubrics, duplicate/orphan prevention, and propose-first governance rules.

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

---

## Included MCP Tools (16 Tools)

| Tool | Required Scope | Description |
| --- | --- | --- |
| `ih_list_products` | `read` | List products and folders accessible to this credential (including `directEdit` status per product). |
| `ih_get_chain` | `read` | Fetch the full User Profile → User Need → PRD → Design → Implementation → Test chain and trace links (`includePending` optional). |
| `ih_get_requirement` | `read` | Fetch one requirement by durable human ID (e.g., `PRD-012`) or UUID, including current revision, design fields, and inbound/outbound links. |
| `ih_get_neighbors` | `read` | Walk upstream (`up`), downstream (`down`), or `both` trace neighbors from any requirement up to `depth`. |
| `ih_search_requirements` | `read` | Search requirement human IDs, titles, and statements with optional `types`, `status`, and `limit` filters. |
| `ih_list_review_queue` | `read` | List proposals in the Review Queue filtered by `status` (`pending`, `accepted`, `rejected`), `source`, or `type`. |
| `ih_validate_against_trace` | `read` | Dry-run validation checking for duplicate titles, missing User Profile, missing parent link, or orphan records without writing. |
| `ih_propose_profile` | `suggest` / `edit` | Add or edit a User Profile (`name`, `roleIntent`). Queues a proposal in `suggest` mode or writes directly in `edit` mode. |
| `ih_propose_requirement` | `suggest` / `edit` | Add a `BN`, `UN`, `PRD`, `SD`, `UXD`, `CD`, `IMPL`, `VT`, `VER`, or `UT` record with optional design/test metadata. |
| `ih_propose_edit` | `suggest` / `edit` | Revise an existing requirement's title, statement, or design/test fields with optional `expectedRevision` optimistic concurrency check. |
| `ih_propose_bulk_push` | `suggest` / `edit` | Validate and atomically apply or queue a batch of requirements and trace links (`dryRun: true` supported). |
| `ih_create_requirement` | `edit` | Direct-only requirement creation on products where the credential has `edit` permission. |
| `ih_update_requirement` | `edit` | Direct-only requirement revision on products where the credential has `edit` permission. |
| `ih_link_requirements` | `suggest` / `edit` | Create a directional `TraceLink` (`addresses`, `realized_by`, `checked_by`, `refined_by`) between two records. |
| `ih_unlink_requirements` | `suggest` / `edit` | Soft-remove a `TraceLink` by `linkId` or by `sourceId` + `type` + `targetId`. |
| `ih_lifecycle_action` | `suggest` / `edit` | Execute lifecycle transitions: `withdraw_proposal`, `obsolete`, `delete_draft`, or `reclassify`. |

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

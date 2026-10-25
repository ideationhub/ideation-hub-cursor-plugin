---
name: ideation-hub-requirements-agent
description: Connect coding agents to Ideation Hub (www.ideationhub.me) via the MCP server at /api/mcp or REST API to read, validate, trace, suggest, or directly edit Business Needs, User Profiles, User Needs, Product Requirements, Designs, Implementations, Tests, and TraceLinks.
version: 1.4.0
---

# Ideation Hub — Requirements & Traceability Agent Skill

**Product:** `https://www.ideationhub.me` · **API:** `https://api.ideationhub.me` · **MCP:** `https://api.ideationhub.me/api/mcp`  
**Audience:** Any coding agent, Cursor agent, or Grok Bot working on a customer's Product in Ideation Hub. Tenant-agnostic: this file holds no secret, tenant id, or product id.

## Design controls basics

Ideation Hub is a requirements workspace built on **design controls**: a traceable chain from business intent to verified product behavior, as used in regulated and AI-first software engineering practice. Each record answers one question; links show how they connect.

- **Business Need** (`BN-NNN`) — a business objective. *Purpose:* why the business invests. *Good one:* one outcome the business wants, measurable where possible, free of solution detail.
- **User Profile** — who the product serves. *Purpose:* anchor needs to a real role. *Good one:* a clear `name` plus a `roleIntent` saying what that role is trying to get done. It declares only the actual person (who they are, their role and context), never their needs; those are User Needs. The API identifies it by UUID or exact name.
  *Rule:* A User Profile must name who the product serves: a role or type of person with a job to do, not a mechanism or a single behavior. If a proposed Profile is really one capability (e.g. "issues scoped agent credentials"), it does not qualify as a Profile; propose that capability as a Product Requirement traced to an existing Profile instead. Proposals that fail this rule should be rejected, with this reasoning given.
- **User Need** (`UN-NNN`) — what one Profile must be able to do. *Purpose:* the user's view of value. *Good one:* "Ability to …", one capability, one Profile; WHAT the user wants to achieve, never HOW (that is the PRDs tracing up to it); it `addresses` the Business Need it serves.
- **Product Requirement** (`PRD-NNN`) — WHAT the product must do. *Purpose:* a buildable, checkable commitment. *Good one:* concise, testable, no who/why or design (see the PRD rubric below); it `addresses` its User Need.
- **Design & Implementation** (`SD-NNN`, `UXD-NNN`, `CD-NNN`, `IMPL-NNN`) — System Design, UX Design (`mockupUrl`), Component Design, and Implementation (`repoPath`, `prUrl`, `commitSha`).
- **Verification & Validation Tests** (`VT-NNN`, `VER-NNN`, `UT-NNN`) — Validation Tests (check UNs), Verification Tests (check PRDs), and Unit Tests (`testPath`, `lastResult`: `pass` | `fail` | `not_run`).
- **Acceptance criteria / verification** — short pass/fail checks on a PRD, each naming an observable result. Verification methods run those checks to show the PRD is met.

**Traceability.** Every child traces up to its parent: PRD → UN → BN. This lets you prove coverage (every BN and UN is served by something), spot gaps, and run impact analysis (what is affected when a record changes).

TraceLink types:

- `addresses` — child → parent (UN addresses BN; PRD addresses UN).
- `refined_by` — a record is refined by a more detailed one.
- `realized_by` — a requirement or design is realized by a downstream design or implementation.
- `checked_by` — a need, requirement, or implementation is checked (verified/validated) by a test record.

Walking **up** answers *why* this exists; walking **down** answers *how* it is delivered and checked. An **orphan** (no parent) or a **gap** (a BN with no UN, a UN with no PRD, a PRD with no checks) is a defect: fix it or flag it.

Every requirement has a **Human Readable Durable ID** (`BN-`, `UN-`, `PRD-`, `SD-`, `UXD-`, `CD-`, `IMPL-`, `VT-`, `VER-`, `UT-` + number, sequenced per type inside a Product) plus a UUID and a revision `@n`. Human Readable Durable IDs never get renumbered or reused.

## Working with the user (conversational workflow)

1. **Have a conversation with the end user or parse a document from them.**
2. **Be interactive and ask for disambiguation of anything you hear.** Pair open questions with suggested interpretations ("Did you mean A or B, or something else?"); never guess.
3. **Disseminate all information from the conversation with the end user** — break everything the user said into candidate requirement elements (BN, Profile, UN, PRD, acceptance criteria, designs, tests), losing nothing.
4. **Search for existing requirements elements: user profiles, user needs, business needs, product requirements, etc.** Use the MCP tools in §3 (`ih_list_products`, `ih_get_chain`, `ih_search_requirements`, `ih_get_requirement`, `ih_get_neighbors`, `ih_validate_against_trace`).
5. **Have a dialogue with the user about what currently exists in the requirements** — matches, overlaps, gaps.
6. **Determine how best to update those requirements or add to them, keeping the design control structure in place:** TraceLinks up to UNs/business needs, the PRD rubric (testable product behavior, no implementation), and propose-only via Review (§4) unless direct edit is explicitly granted. Apply the layered pattern (see Writing rules): Profile = the person, UN = the WHAT, PRD = the HOW as testable product behavior.
7. **Recommend, don't just transcribe.** If the user is misguided (e.g. needs in a Profile, a HOW in a User Need), still recommend how to define the requirements clearly: explain the better structure and offer a rewritten version.

## 1. Bot credentials (guide the user)

The user may not know how to do this. Relay these steps in plain words:

1. Open the setup link `https://www.ideationhub.me/settings/agents?setup=grokbot` (a `next` query param is kept through sign-in) and sign in as an **Admin** of the Product.
2. On **Agent credentials** (`/settings/agents`; also Product → Agents) choose **Create**.
3. Enter a **Client ID**: letters, digits, `.` `_` `-`, starting with a letter, no spaces (e.g. `GrokBot`; `Grok Bot` fails). Pattern `^[A-Za-z][A-Za-z0-9._-]{2,63}$`.
4. Pick the permission for this product:
   - **Read** — `scopes: ["read"]`, `editDirectlyProductIds: []`. The bot can look, not write.
   - **Suggest** — `scopes: ["read", "propose"]`, `editDirectlyProductIds: []`. Writes land in the issuer's Review Queue (`/api/me/requirements-review`) until a human Accepts. `suggest` is the same permission as `propose`.
   - **Edit** — `scopes: ["read", "propose"]` and `editDirectlyProductIds` set to the products this bot may change directly (the "Allow this agent to edit this product directly" checkbox), or an `edit` scope on those products. Writes apply to the living record unless the call passes `mode: "suggest"` or `"propose"`.
5. Copy the **Client ID** and **Client Secret** from the masked secret card right away — the secret is shown **only once**.
6. Paste them into the plugin / connector setup dialog (`IH_CLIENT_ID` and `IH_CLIENT_SECRET`). Never ask for secrets in plain chat text.
7. Confirm the credential with `ih_list_products` (or `initialize` on `/api/mcp`, §3), echoing nothing.

- **Lost secret:** create a new credential or rotate it on the same screen, then repeat step 5.
- **Issuer:** the Admin who created the credential is the issuer and Accepts or Rejects your proposals.

**Secrets rule:** never paste, echo, log, or print the secret or an access token — not in chat, Git, PRs, docs, tickets, or URLs. Reference env vars only. If a secret leaks, tell the human to rotate or revoke on the same www screen.

## 2. Authentication (MCP Headers or OAuth Token)

### Direct MCP Headers (Preferred)
Pass `X-IH-Client-Id: ${IH_CLIENT_ID}` and `X-IH-Client-Secret: ${IH_CLIENT_SECRET}` directly to `https://api.ideationhub.me/api/mcp` (configured automatically by `mcp.json`).

### OAuth 2.0 `client_credentials` Token (Optional / REST)
`POST /api/oauth/token` (also `/api/v0/...`; every route below is dual-mapped):

```json
{ "grant_type": "client_credentials", "client_id": "$IH_CLIENT_ID",
  "client_secret": "$IH_CLIENT_SECRET", "productId": "<uuid, optional>" }
```

- **200** `{ access_token, token_type: "Bearer", expires_in, productId }`. Omit `productId` for the credential's home Product; a sent one must be on its allow-list.
- **401** wrong secret / revoked. Re-mint when the token expires.
- Send `Authorization: Bearer <token>`, or call MCP with `X-IH-Client-Id` and `X-IH-Client-Secret` and skip the token. The token pins tenant + Product. MCP may pass `productId` only when that product is on the credential grant; otherwise **403**.
- Permissions: `read`, `suggest` (stored as `propose`), and direct edit via `editDirectlyProductIds` or an `edit` scope. Missing permission → **403**. `review` is not a scope.

## 3. MCP Tools (Preferred — 16 Tools at `POST /api/mcp`)

Streamable HTTP JSON-RPC at `POST https://api.ideationhub.me/api/mcp` (also `/api/v0/mcp` and `/mcp`). No SSE required. `GET` returns server metadata. Headers: `X-IH-Client-Id`, `X-IH-Client-Secret` (or `Authorization: Bearer <token>`).

`initialize` returns protocol `2025-03-26` (or `2024-11-05` when requested), server `ideation-hub-mcp`. `tools/list` and `tools/call` require an agent credential. Omit `productId` to use the credential's home product. A product outside the grant returns **403**. `ih_list_products` returns only granted products.

| Tool | Permission | Parameters & Behavior |
| --- | --- | --- |
| `ih_list_products` | `read` | `{folderId?}` — Granted products (`directEdit` boolean per product) and visible folders. |
| `ih_get_chain` | `read` | `{productId?, includePending?}` — Profiles, requirements, and trace links on the product. `includePending` keeps unapproved rows. |
| `ih_search_requirements` | `read` | `{query, types?, status?, limit?, productId?}` — Search human id, title, and statement on granted products. |
| `ih_get_requirement` | `read` | `{id, productId?}` — One requirement by human id (`PRD-001`) or UUID, including revision, design fields, and inbound/outbound links. |
| `ih_get_neighbors` | `read` | `{id, direction?: "up"\|"down"\|"both", depth?, productId?}` — Walk trace neighbors upstream or downstream. |
| `ih_validate_against_trace` | `read` | `{id?, proposedPayload?, productId?}` — Duplicate title, missing profile, missing parent, and orphan checks. Does not write. |
| `ih_list_review_queue` | `read` | `{productId?, type?, source?: "all"\|"agent"\|"human"\|"mine", status?: "pending"\|"accepted"\|"rejected"}` — Proposals visible to this credential. |
| `ih_propose_profile` | `suggest` or `edit` | `{action: "add"\|"edit", name?, roleIntent?, id?, mode?: "suggest"\|"propose"\|"edit", productId?}` — Add or edit a User Profile. Direct edit writes the living profile; suggest queues it. |
| `ih_propose_requirement` | `suggest` or `edit` | `{type: "BN"\|"UN"\|"PRD"\|"SD"\|"UXD"\|"CD"\|"IMPL"\|"VT"\|"VER"\|"UT", statement, title?, profileNameOrId?, parentId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, mode?, productId?}` — Add a requirement. See Write Rule below. |
| `ih_propose_edit` | `suggest` or `edit` | `{id, title?, statement?, expectedRevision?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, mode?, productId?}` — Revise a requirement. `expectedRevision` mismatch is **409**; no-op is **422**. |
| `ih_propose_bulk_push` | `suggest` or `edit` | `{items, links?, dryRun?, mode?, productId?}` — Validate every item, then apply or queue them together. `dryRun` does not persist or consume human ids. |
| `ih_create_requirement` | `edit` only | Direct-only add (same fields as `ih_propose_requirement`). Returns **403** unless the product is in `editDirectlyProductIds` or credential has `edit` scope. |
| `ih_update_requirement` | `edit` only | Direct-only revise (same fields as `ih_propose_edit`). Returns **403** without direct edit on target product. |
| `ih_link_requirements` | `suggest` or `edit` | `{sourceId, type: "addresses"\|"realized_by"\|"checked_by"\|"refined_by", targetId, productId?}` — Create a trace link directly (not queued as a proposal). |
| `ih_unlink_requirements` | `suggest` or `edit` | `{linkId?, sourceId?, type?, targetId?, productId?}` — Soft-remove a trace link by `linkId` or by `sourceId` + `type` + `targetId`. |
| `ih_lifecycle_action` | `suggest` or `edit` | `{id, action: "withdraw_proposal"\|"obsolete"\|"delete_draft"\|"reclassify", targetType?, reason?, mode?, productId?}` — Lifecycle actions on a requirement or proposal. |

**Write rule.** If the credential has direct edit on the target product (`editDirectlyProductIds` or `edit` scope) and the call does not pass `mode: "suggest"` or `"propose"`, the change is applied to the living record and audited with the agent client id and the issuer user id. Otherwise it is a pending proposal (`proposalId`, `reservedHumanId`, `status: "pending"`, `reviewUrl`) on the issuer's Review Queue. Pass `mode: "suggest"` to queue even when direct edit is allowed. `mode: "edit"` without direct edit is **403**. Agents cannot Accept or Reject (**403**).

Design fields on UXD / IMPL / VT / VER / UT: `mockupUrl`, `repoPath`, `prUrl`, `commitSha`, `testPath`, `lastResult` (`pass`, `fail`, `not_run`; `passed` and `failed` are accepted). A User Need takes `profileNameOrId`. A Product Requirement takes `parentId` (or `unId`) and `addresses` that User Need.

## 4. Propose a change (workflow & REST fallback)

Workflow before any write:

1. **Read neighbors first.** Pull the chain (`ih_get_chain`) or `ih_get_neighbors` for the parent you will attach to.
2. **Check for duplicates.** Search existing titles/statements (`ih_search_requirements`); prefer editing an existing record over adding a near-copy.
3. **Validate & propose with a parent.** Use `ih_validate_against_trace` or `ih_propose_bulk_push` (`dryRun: true`) when staging multiple changes.

REST endpoints remain available and follow the same read / suggest / edit split:

| Need | REST Call |
| --- | --- |
| All requirements on the Product | `GET /api/requirements` |
| One requirement | `GET /api/requirements/{id}` (human id or UUID) |
| Neighbors up/down the trace | `GET /api/requirements/{id}/neighbors?direction=up\|down` |
| TraceLinks | `GET /api/trace-links` · `POST /api/trace-links` |
| Profiles | `GET /api/products/{productId}/profiles` · `POST /api/products/{productId}/profiles` · `PATCH /api/profiles/{id}` |
| Full Profile → UN → PRD chain | `GET /api/products/{productId}/chain` (or `GET /api/chain`) |
| Add / Edit / Lifecycle | `POST /api/requirements` · `POST /api/requirements/{id}/revisions` · `POST /api/requirements/{id}/obsolete` · `POST /api/requirements/{id}/unaccept` · `POST /api/requirements/{id}/reclassify` · `POST /api/requirements/{id}/delete` · `POST /api/requirements/{id}/restore` · `POST /api/requirements/{id}/withdraw` |

## 5. Report back to the human

After proposing, tell the issuer in one short message:

- Each proposal `id` (or `proposalId`), `kind` (add/edit/obsolete/…), and the target Human Readable Durable ID or title. A TraceLink is not a proposal: report its link id.
- Where to decide: **Review changes** on www (`https://www.ideationhub.me`, avatar → *Agent proposals*, or the Product's Review tab).
- Anything rejected with **422** and why. Never include secrets or tokens.

## Writing rules (PRD rubric)

- A PRD states only **WHAT the product must do**, concisely. A checker can pass/fail it without the linked UN.
- No who / why / role / purpose / outcome — the TraceLinks up to the UN carry that.
- No component counts, UI surfaces (pages, dialogs, controls), implementation shape, or design/engineering decisions.
- Acceptance criteria: each one short pass/fail check naming an observable result.
- No IDs in prose and no "extends PRD-…" in statements. Express relationships as TraceLinks.
- User Needs read "Ability to …" and state the user's objective (WHAT), not HOW it is delivered; PRDs that trace up to the UN carry the HOW.
- A User Profile declares only the person (who, role, context), not their needs; capture those as User Needs.
- Every User Need must trace up to one or more User Profiles. A User Need with no Profile is an orphan: link it to the Profile(s) it serves, or flag it.
- A User Profile must name who the product serves: a role or type of person with a job to do, not a mechanism or a single capability.
- **Layered pattern — apply on every create or update:** User Profiles declare only the person; User Needs declare only the WHAT (the user's objectives); Product Requirements declare the HOW, each tracing up to its User Need. The PRD's "how" is the product capability that satisfies the need, stated as testable product behavior, never implementation or design.

## Business Need vs User Need (rubric)

- A **Business Need** is what the business that owns the product must accomplish for itself: revenue, cost, compliance, strategy, or operations. It is not for the sake of the customer.
- Anything meant to help a customer or user is a **User Need**, traced up to the relevant customer User Profile.

Pass/fail checks when filing or reviewing:

1. **Who benefits directly?** The owning business → Business Need. A customer or user → User Need.
2. **Can you name the customer Profile it serves?** If yes, it is a User Need.
3. A Business Need statement contains no customer benefit or outcome.
4. Reviewers flag any misfiled item and re-propose it as the right type; never delete it.

## Anti-patterns

- Echoing a secret or token anywhere
- Writing without reading neighbors, or adding duplicates
- Orphan PRDs (no parent UN). A TraceLink may name a still-pending requirement; it stays hidden until both ends are canonical.
- Asking for a `review` scope (**422**) or trying to Accept your own proposals
- Writing a living record from a Suggest credential, or from an Edit credential when the call says `mode: "suggest"` — that change waits in Review
- Editing a product that is not in `editDirectlyProductIds` and not covered by `edit` scope

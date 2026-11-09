---
name: ideation-hub-requirements-agent
description: Connect coding agents to Ideation Hub (www.ideationhub.me) via the MCP server at /api/mcp or REST API to read, validate, trace, suggest, or directly edit Business Needs, User Profiles, User Needs, Product Requirements, Designs, Implementations, Tests, and TraceLinks.
version: 1.4.9
---

# Ideation Hub — Requirements & Traceability Agent Skill

**Product:** `https://www.ideationhub.me` · **API:** `https://api.ideationhub.me` · **MCP:** `https://api.ideationhub.me/api/mcp`  
**Audience:** Any coding agent, Cursor agent, or Grok Bot working on a customer's Product in Ideation Hub. Tenant-agnostic: this file holds no secret, tenant id, or product id.

## Design controls basics

Ideation Hub is a requirements workspace built on **design controls**: a traceable chain from business intent to verified product behavior, as used in regulated (e.g. medical device) and ALM practice. Each record answers one question; links show how they connect.

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
- `realized_by` — a need is realized by a requirement, or a requirement or design by a downstream design or implementation.
- `checked_by` — a need, requirement, or implementation is checked (verified or validated) by another record, such as a test.

Walking **up** answers *why* this exists; walking **down** answers *how* it is delivered and checked. An **orphan** (no parent) or a **gap** (a BN with no UN, a UN with no PRD, a PRD with no checks) is a defect: fix it or flag it.

Every requirement has a **Human Readable Durable ID** (`BN-`, `UN-`, `PRD-`, `SD-`, `UXD-`, `CD-`, `IMPL-`, `VT-`, `VER-`, `UT-` + number, sequenced per type inside a Product) plus a UUID and a revision `@n`. Human Readable Durable IDs never get renumbered or reused.

## Working with the user (conversational workflow)

1. **Have a conversation with the end user or parse a document from them.**
2. **Be interactive and ask for disambiguation of anything you hear.** Pair open questions with suggested interpretations ("Did you mean A or B, or something else?"); never guess.
3. **Disseminate all information from the conversation with the end user** — break everything the user said into candidate requirement elements (BN, Profile, UN, PRD, acceptance criteria, designs, tests), losing nothing.
4. **Search for existing requirements elements: user profiles, user needs, business needs, product requirements, etc.** Use the MCP tools in §3 (`ih_list_products`, `ih_get_chain`, `ih_search_requirements`, `ih_get_requirement`, `ih_get_neighbors`, `ih_validate_against_trace`) or the REST read endpoints there.
5. **Have a dialogue with the user about what currently exists in the requirements** — matches, overlaps, gaps.
6. **Determine how best to update those requirements or add to them, keeping the design control structure in place:** TraceLinks up to UNs/business needs, the PRD rubric (testable product behavior, no implementation), and propose-only via Review (§4) unless direct edit is explicitly granted. Apply the layered pattern (see Writing rules): Profile = the person, UN = the WHAT, PRD = the HOW as testable product behavior.
7. **Recommend, don't just transcribe.** If the user is misguided (e.g. needs in a Profile, a HOW in a User Need), still recommend how to define the requirements clearly: explain the better structure and offer a rewritten version.

## Upward Traceability & Discovery Guidelines (Required When Authoring Requirements)

1. **Every PRD must trace up to one or more User Needs (`PRD --addresses--> UN`).** A PRD with no upward trace to a User Need is an orphan defect; link it to every User Need it satisfies via `parentId` on create and/or `ih_link_requirements` (`sourceId: "<PRD-ID>", type: "addresses", targetId: "<UN-ID>"`).
2. **Search existing User Needs whenever defining a new requirement.** Before creating or updating a requirement, search all existing User Needs in the Product (`ih_search_requirements` or `ih_get_chain`) and consider which existing User Needs that new requirement satisfies.
3. **Suggest candidate new User Needs or Business Objectives and ask before creating them.** Also consider which *new* User Needs or Business Objectives (Business Needs) the new requirement would satisfy that are not yet in the Product. Suggest those to the user as candidate proposals and ask whether they are reasonable before creating them.

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
6. Paste them into the bot's credentials dialog (secure secret request) or the plugin / connector setup dialog, as `IH_CLIENT_ID` and `IH_CLIENT_SECRET`. A bot opens that dialog itself. Never ask for secrets in chat.
7. The bot confirms the credential with `initialize` on `/api/mcp` or `ih_list_products` (§3), echoing nothing.

- **Lost secret:** create a new credential or rotate it on the same screen, then repeat step 5.
- **Issuer:** the Admin who created the credential is the issuer and Accepts or Rejects your proposals.

**Secrets rule:** never paste, echo, log, or print the secret or an access token — not in chat, Git, PRs, docs, tickets, or URLs. Reference env vars only. If a secret leaks, tell the human to rotate or revoke on the same www screen.

## 2. Authentication (MCP headers or OAuth token)

### Direct MCP headers (preferred)

Pass `X-IH-Client-Id: ${IH_CLIENT_ID}` and `X-IH-Client-Secret: ${IH_CLIENT_SECRET}` directly to `https://api.ideationhub.me/api/mcp` (the plugin's `mcp.json` configures this automatically).

### OAuth 2.0 `client_credentials` token (optional / REST)

`POST /api/oauth/token` (also `/api/v0/...`; every route below is dual-mapped):

```json
{ "grant_type": "client_credentials", "client_id": "$IH_CLIENT_ID",
  "client_secret": "$IH_CLIENT_SECRET", "productId": "<uuid, optional>" }
```

- **200** `{ access_token, token_type: "Bearer", expires_in, productId }`. Omit `productId` for the credential's home Product; a sent one must be on its allow-list.
- **401** wrong secret / revoked. Re-mint when the token expires.
- Send `Authorization: Bearer <token>`, or call MCP with `X-IH-Client-Id` and `X-IH-Client-Secret` and skip the token. The token pins tenant + Product. MCP may pass `productId` only when that product is on the credential grant; otherwise **403**.
- Permissions: `read`, `suggest` (stored as `propose`), and direct edit via `editDirectlyProductIds` or an `edit` scope. Missing permission → **403**. `review` is not a scope.

## 3. MCP (preferred)

Streamable HTTP JSON-RPC at `POST https://api.ideationhub.me/api/mcp` (also `/api/v0/mcp` and `/mcp`). No SSE. `GET` returns server metadata. Headers: `X-IH-Client-Id`, `X-IH-Client-Secret`. A bearer token from §2 also works.

`initialize` returns protocol `2025-03-26` (or `2024-11-05` when requested), server `ideation-hub-mcp`. `tools/list` and `tools/call` require an agent credential. Omit `productId` to use the credential's current product. A product outside the grant is **403**. `ih_list_products` returns only granted products.

| Tool | Permission | What it does |
| --- | --- | --- |
| `ih_list_products` | read | `{folderId?}`. Granted products (`directEdit` per product) and visible folders |
| `ih_get_chain` | read | `{productId?, includePending?, tag?, tags?}`. Profiles, requirements, and links. `includePending` keeps unapproved rows. Optional `tag` or `tags` (AND, case-insensitive) |
| `ih_search_requirements` | read | `{query, types?, status?, productId?, limit?, tag?, tags?}`. `query` is required. Tag filters are AND |
| `ih_get_requirement` | read | `{id, productId?}`. Human id (`PRD-001`) or UUID, revision, design fields, tags, inbound and outbound links |
| `ih_get_neighbors` | read | `{id, direction?: up\|down\|both, depth?, productId?}` |
| `ih_validate_against_trace` | read | `{id?, proposedPayload?, productId?}`. Duplicate title, missing profile or parent, orphan. Does not write |
| `ih_list_review_queue` | read | `{status?, source?: all\|agent\|human\|mine, type?, productId?, tag?, tags?}`. Each item's tags merge the payload and the target requirement |
| `ih_propose_requirement` | suggest or edit | `{type, statement, title?, profileNameOrId?, parentId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, tags?, mode?, productId?}`. Add BN/UN/PRD/SD/UXD/CD/IMPL/VT/VER/UT. See the write rule below |
| `ih_propose_edit` | suggest or edit | `{id, title?, statement?, expectedRevision?, proposalId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, tags?, mode?, productId?}`. Revise. One own pending add or edit for that requirement is amended in place (`amended: true`, same `proposalId`). A pending add updates the unapproved draft. A pending edit does not change an accepted requirement. Pass `proposalId` when more than one of yours is pending (`409` `ambiguous_proposal`). Another author's proposal is **403** `not_proposal_author`. Optional `tags`. `expectedRevision` mismatch is **409**. A no-op is **422** |
| `ih_propose_bulk_push` | suggest or edit | `{items, links?, dryRun?, mode?, productId?}`. `dryRun` does not persist or consume human ids |
| `ih_propose_profile` | suggest or edit | `{action: add\|edit, id?, name, roleIntent, mode?, productId?}` |
| `ih_create_requirement` / `ih_update_requirement` | edit only | Direct aliases. Optional `tags`. Suggest-only is **403** |
| `ih_set_tags` | edit only | `{id, tags, productId?}`. Replaces tags. Does not create a revision. Suggest-only is **403** |
| `ih_link_requirements` | suggest or edit | `{sourceId, type: addresses\|realized_by\|checked_by\|refined_by, targetId, productId?}`. Add a TraceLink. Edit applies it (`outcome: applied`, `link`). Suggest bundles it with a pending add when exactly one end is that new requirement (`outcome: bundled`, `bundledWith`). Two pending adds stay one pending proposal and are not their own Review row until both ends are approved. A link between existing requirements is `outcome: proposed` |
| `ih_unlink_requirements` | suggest or edit | `{linkId?, sourceId?, type?, targetId?, productId?}`. Soft-remove by `linkId` or source + type + target. Same applied-versus-proposed split. Reject or withdraw leaves the link |
| `ih_lifecycle_action` | suggest or edit | `{id, action, targetType?, reason?, mode?, productId?}`. Actions: `withdraw_proposal` (proposal id only), `withdraw_requirement`, `reopen_proposal` (withdrawn add-link at version 1; creator, withdrawing actor, or editor), `reopen_requirement` (edit), `obsolete`, `delete_draft`, `reclassify` |
| `ih_restore_draft_title` | suggest | Title-only repair of the caller's pending add. Full snapshot required. Does not change the proposal |

**Write rule.** If the credential has direct edit on the target product (`editDirectlyProductIds` or `edit` scope) and the call does not pass `mode: "suggest"` or `"propose"`, the change is applied to the living record and audited with the agent client id and the issuer user id. Otherwise it is a pending proposal (`proposalId`, `reservedHumanId`, `status: "pending"`, `reviewUrl`) on the issuer's Review Queue. A TraceLink with exactly one pending-add end is the exception in §4: it is stored with that requirement instead of a standalone proposal. Pass `mode: "suggest"` to queue even when direct edit is allowed. Two pending adds stay one pending proposal and are hidden from standalone Review until both ends are approved. `mode: "edit"` without direct edit is **403**. Agents cannot Accept or Reject (**403**).

Design fields on UXD / IMPL / VT / VER / UT: `mockupUrl`, `repoPath`, `prUrl`, `commitSha`, `testPath`, `lastResult` (`pass`, `fail`, `not_run`; `passed` and `failed` are accepted). A User Need takes `profileNameOrId`. A Product Requirement takes `parentId` or `unId` and `addresses` that User Need.

Tags are a free-form list (`tags: string[]`). Each tag is 1–32 characters from letters, numbers, spaces, and `_-./`, at most 20, trimmed and de-duplicated without regard to case. Omit `tags` to leave them unchanged. `[]` clears them. Responses always include `tags` (an empty list when there are none). `ih_set_tags` and `PUT /api/requirements/{id}/tags` replace the list without a new revision. Review items also show tags stored on the target requirement, so a pending proposal can be filtered by a tag that was seeded on that row.

REST below remains available and follows the same read / suggest / edit split.

| Need | Call |
| --- | --- |
| All requirements on the Product | `GET /api/requirements` |
| One requirement | `GET /api/requirements/{id}` (human id or UUID) |
| Neighbors up/down the trace | `GET /api/requirements/{id}/neighbors?direction=up\|down` |
| TraceLinks | `GET /api/trace-links` |
| Profiles | `GET /api/products/{productId}/profiles`, `GET /api/profiles/{id}` |
| Full Profile → UN → PRD chain | `GET /api/products/{productId}/chain` (or `GET /api/chain`) |

## 4. Propose a change (write)

A **Suggest** credential (`read` + `propose`, no direct edit on that product) files a **proposal**. It lands in **Review**; the living record stays `unapproved` until the credential issuer Accepts. An **Edit** credential (`editDirectlyProductIds` for that product, or `edit` scope) writes the living record unless the call passes `mode: "suggest"` or `"propose"`. Agents cannot Accept or Reject (**403**).

**TraceLink review (partial).** The shipped rule covers an add-link with exactly one pending-add end. It is not the complete rule. When exactly one end is a pending add (`unapproved`, add still pending), a Suggest link is stored with that requirement (`outcome: "bundled"`, `bundledWith` is its Human Readable Durable ID). There is no standalone Review item. The link shows on that requirement's Review card, upstream or downstream. Accept keeps it and records both directions as `approvedLinks`. Reject or withdraw of the add hides it. This is either direction: pending source to an existing target, or an existing source to a pending target. An accepted requirement stays existing when it only has a pending edit. A link between two existing requirements is still its own Review item (`outcome: "proposed"`, `proposalId`). Accept applies that add or the soft-remove. Reject and withdraw leave that trace unchanged. Direct edit still writes the row (`outcome: "applied"`). Read-only is **403**. Removing a link is unchanged: Suggest files a removal proposal, and direct edit soft-removes it. A link between two pending adds is not bundled and is not withdrawn to hide it. It stays one pending proposal, omitted from standalone Review while either end is still a pending add, and listed on those requirements. Accepting one end does not approve the other or write the trace. Accepting the link while an end is pending is **409** `ends_pending`. The same proposal is accepted and the trace is written when the last pending end is accepted. Do not treat a removal as part of that rule.

Soft delete is a proposal for Suggest, and applies immediately when that credential has direct edit.

Workflow:

1. **Read neighbors first.** Pull the chain (`ih_get_chain`) or neighbors (`ih_get_neighbors`) for the parent you will attach to.
2. **Check duplicates, conflicts, and related requirements before writing.** Search existing requirements and pending proposals by meaning as well as title and statement, including their profiles, scope, status, and traces. Prefer updating an appropriate existing requirement over creating a duplicate. For every proposed new or updated requirement, evaluate other related requirements and all appropriate upward trace targets; link only where the proposed capability genuinely satisfies the need, not merely because wording is similar. Preserve distinctions such as tenant-wide versus product-specific authority.
   - **Clarify ambiguous definitions or structure before any write.** If duplicate, conflicting, overlapping, or related requirements leave ambiguity about which record to create or update, whether to combine or separate capabilities, or which traces are appropriate, explain the relevant records and recommend a structure, then ask the user to resolve that ambiguity. Do not silently merge, broaden scope, withdraw, or invent relationships. Clear, non-conflicting matches do not require another confirmation. A second `add` of an existing key conflicts.
3. **Consider traces whenever modifying a requirement.** Before any edit, withdrawal, obsolete, reclassify, or link change, read the requirement's inbound and outbound traces (`ih_get_requirement`, `ih_get_neighbors`). Check whether each existing trace still holds after the change, whether any should be added, removed, or redirected, and whether linked requirements need their own updates. Where you are not certain a trace or related change is right, ask the user for guidance rather than guessing.
4. **Explain planned related changes and ask before acting on recommendations.** Before writing, tell the user the main change and every related change you plan (trace additions or removals, edits to linked requirements, withdrawals), with a one-line rationale for each, and ask any open questions. Proceed only with what the user confirms; changes the user explicitly asked for need no second confirmation, but related changes you recommend do.
5. **Validate and propose with a parent.** Use `ih_validate_against_trace`, or `ih_propose_bulk_push` with `dryRun: true` when staging several changes, before writing.

| Change | Call | Notes |
| --- | --- | --- |
| Add BN/UN/PRD | `POST /api/requirements` | `type`: `business_need`\|`user_need`\|`product_requirement`; `title`, `statement`. UN needs `profileId` or a `profileName` matching exactly one canonical Profile. PRD needs `unId` of a canonical UN; Accept writes the `addresses` link. |
| Edit | `POST /api/requirements/{id}/revisions` | `{ title?, statement?, proposalId? }`. The caller's one pending add or edit for that requirement is updated in place. A new edit proposal is filed only when the caller has none pending. |
| Link | `POST /api/trace-links` | Suggest, exactly one pending-add end: stored with that requirement (`outcome: "bundled"`, `bundledWith`). No standalone Review item. Suggest, both ends existing: `kind=link` proposal. Direct edit writes the row. Two pending adds stay a standalone proposal. |
| Obsolete | `POST /api/requirements/{id}/obsolete` | |
| Un-accept | `POST /api/requirements/{id}/unaccept` | `{ targetRevision?, reason? }`. Accepted requirements only (status `proposed`). Anything else is **4xx** and nothing is stored. Moves that requirement back to pending (`unapproved`) and keeps its Human Readable Durable ID — no renumbering. When `targetRevision` is an earlier `@n`, that revision's title and statement are copied onto a **new** revision. History is not rewritten. TraceLinks stay. |
| Reclassify | `POST /api/requirements/{id}/reclassify` | `{ newType, reason? }`. Accepted (`proposed` or `approved`) and pending (`unapproved`) only; otherwise **422**. Assigns the next human id for `newType` and keeps the old id as an alias. |
| Delete draft | `DELETE /api/requirements/{id}` | Only living status `proposed`; else **422**. This is the hard-delete path and stays refused for humans (`noHardDelete`). |
| Soft delete | `POST /api/requirements/{id}/delete` | `{ reason? }`. Sets `deleted_at`, `deleted_by`, and `deleted_reason`. The row, human id, and TraceLinks stay. The requirement leaves lists, Review, and trace views; the links stay stored and stay hidden. Human ids are not reused. Product owners, tenant admins, and `edit_directly` holders apply it immediately. An agent with only the propose scope files a `soft_delete` proposal; Accept applies it. **409** `already_deleted` when it is already deleted. |
| Restore | `POST /api/requirements/{id}/restore` | Clears the soft delete and brings the requirement and its TraceLinks back. Same direct callers as soft delete. Propose-scope agents do not file a restore proposal. **409** `not_deleted` when it is still live. |
| Withdraw one proposal | `POST /api/proposals/{id}/withdraw` | `{ reason? }`. `id` is the proposal id. Closes that pending proposal only. A requirement human id or uuid is **422** `requirement_id` and changes nothing. |
| Reopen one add-link proposal | `POST /api/proposals/{id}/reopen` | `{ reason? }`. `id` is a withdrawn `kind=link` add proposal at version `1`. Same id, payload, and version. No trace is written. A removal is **422** `link_removal`. Any other status or version is **409** `not_withdrawn_v1`. Direct edit, product owner, or tenant admin. Suggest scope is **403**. `reopen_requirement` does not do this. |
| Withdraw pending requirement | `POST /api/requirements/{id}/withdraw` | `{ reason? }`. Only `unapproved`. Soft-removes the requirement and closes every pending proposal on it. The row and human id stay. **409** otherwise. MCP action: `withdraw_requirement`. |
| Reopen withdrawn requirement | `POST /api/requirements/{id}/reopen` | `{ reason? }`. Only `withdrawn`. Sets the requirement back to `unapproved` and reopens its add proposal. Other proposals stay withdrawn. Direct edit, product owner, or tenant admin. Propose-scope agents get **403**. |
| Profile add / edit | `POST /api/products/{productId}/profiles` · `PATCH /api/profiles/{id}` | `{ name, roleIntent }`; never invent `roleIntent`. |

- Failed propose → **422** (unknown `type` → **400**); no proposal is stored. A pending UN or Profile is not a valid parent — wait for Accept, then propose the child.
- **Un-accept and the same rule as edit and obsolete.** A propose-scope agent files a Review proposal. A human editor with `X-Write-Mode: direct`, or an agent whose credential has edit-directly on this Product, applies it immediately. Either way the applied change writes an audit entry with the actor, the before state, the after state, and a timestamp. Requirements are never hard-deleted.
- Reclassify follows the same propose versus edit-directly rule as edit and obsolete. A propose-scope agent files a Review proposal. A human editor, or edit-directly mode (`X-Write-Mode: direct` with the edit_directly grant, or an agent credential that has edit_directly), applies it immediately. The write records an audit entry with the actor, the requirement before, the requirement after, and a timestamp. The row is never hard-deleted.
- The old human id still resolves (`GET /api/requirements/BN-038`). The UI shows `formerly BN-038`. TraceLinks stay on the same row. Links whose direction no longer matches the new type are reported as `directionInvalidLinks` and are not dropped.
- **No-op edits:** do not propose an edit whose title and statement match the current revision. It adds Review noise and changes nothing.
- Never re-propose to dodge a Reject; ask the human.
- **Withdraw one proposal.** `ih_lifecycle_action` with `action: "withdraw_proposal"` takes a **proposal id**. It closes that proposal and leaves the requirement and every other proposal alone. A requirement human id or uuid is **422** `requirement_id` and changes nothing. `action: "reopen_proposal"` is the inverse for one withdrawn add-link at version 1. Same proposal id and payload. It does not write a trace. A removal is rejected. The creating credential, the credential recorded on the withdraw audit, or an editor may call it. A second distinct withdraw actor is a conflict and is refused.
  - To withdraw the pending requirement and every pending proposal on it, the user must ask for that explicitly. Then use `action: "withdraw_requirement"` or `POST /api/requirements/{id}/withdraw`.
  - To undo that, an editor uses `action: "reopen_requirement"` or `POST /api/requirements/{id}/reopen`. The requirement returns to pending and its add proposal reopens. Edit proposals stay withdrawn. Propose-scope agents cannot reopen.
  - *Example (PRD-038, 2026-10-30):* an agent passed `id: "PRD-038"` to `withdraw_proposal` when that action still treated the id as a requirement. The whole pending requirement closed. That call is now rejected. Pass the proposal id to withdraw one proposal.

## 5. Report back to the human

After proposing, tell the issuer in one short message:

- Each proposal `id`, `kind` (add/edit/obsolete/…), and the target Human Readable Durable ID or title. For a TraceLink, report the link id when the result is `bundled` or `applied`, or the `proposalId` when it is a standalone review between two existing requirements, and say which of those it is.
- Where to decide: **Review changes** on www (`https://www.ideationhub.me`, avatar → *Agent proposals*, or the Product's Review tab).
- Anything rejected with **422** and why. Never include secrets or tokens.

## Writing rules (PRD rubric)

- **Limit each body to that requirement's purpose.** On a new requirement or a revision, include only the detail that states that requirement. Omit background and explanatory subject detail that is not needed to state it. A separate capability belongs in its own traced requirement.
  - Example of this rule, not a change to a living record: a requirement whose purpose is an optional per-project STPA layer states that per-project on/off capability. It does not enumerate what the STPA layer contains or explain STPA.
- A PRD states only **WHAT the product must do**, concisely. A checker can pass/fail it without the linked UN.
- No who / why / role / purpose / outcome — the TraceLinks up to the UN carry that.
- No component counts, UI surfaces (pages, dialogs, controls), implementation shape, or design/engineering decisions.
- Acceptance criteria: each one short pass/fail check naming an observable result.
- No IDs in prose and no "extends PRD-…" in statements. Express relationships as TraceLinks.
- User Needs read "Ability to …" and state the user's objective (WHAT), not HOW it is delivered; PRDs that trace up to the UN carry the HOW.
- **No user-story form in User Needs.** Never write "As a <role>, I need …" or otherwise restate who the user is; the User Profile trace already carries the role, so it is redundant. State only the objective, simply, without mechanisms such as credentials, permission models, CRUD lists, or approval workflow (those belong in the PRDs beneath).
  - *Example (UN-097, 2026-11-09):* Bad: "As a Product Owner enabling agents, I need an agent with Product-scoped credentials and RBAC to Create, Read, Update, and Delete requirements on that Product via MCP, with Create/Update/Delete as proposals only." Good: "Create an account for agents to read and propose requirement changes to a product I own."
- A User Profile declares only the person (who, role, context), not their needs; capture those as User Needs.
- Every User Need must trace up to one or more User Profiles. A User Need with no Profile is an orphan: link it to the Profile(s) it serves, or flag it.
- A User Profile must name who the product serves: a role or type of person with a job to do, not a mechanism or a single behavior. If a proposed Profile is really one capability (e.g. "issues scoped agent credentials"), it does not qualify as a Profile; propose that capability as a Product Requirement traced to an existing Profile instead. Proposals that fail this rule should be rejected, with this reasoning given.
- **Layered pattern — apply on every create or update:** User Profiles declare only the person; User Needs declare only the WHAT (the user's objectives); Product Requirements declare the HOW, each tracing up to its User Need. The PRD's "how" is the product capability that satisfies the need, stated as testable product behavior, never implementation or design.

**Bad:** "MCP server at /api/mcp with existing agent surface plus three tools." (count + implementation)  
**Good:** "On one Product, requirement titles and statements can be searched …" with checks like "Pass: search returns a matching requirement and none from another Product."

## Business Need vs User Need (rubric)

- A **Business Need** is what the business that owns the product must accomplish for itself: revenue, cost, compliance, strategy, or operations. It is not for the sake of the customer.
- Anything meant to help a customer or user is a **User Need**, traced up to the relevant customer User Profile.

Pass/fail checks when filing or reviewing:

1. **Who benefits directly?** The owning business → Business Need. A customer or user → User Need.
2. **Can you name the customer Profile it serves?** If yes, it is a User Need.
3. A Business Need statement contains no customer benefit or outcome.
4. Reviewers flag any misfiled item and re-propose it as the right type; never delete it.

Examples:

- BN-002 "Delivery speed is visible and measurable" is misfiled. As a User Need (Profile: Product Owner): "Ability to see how long my products take to build."
- BN-038 "Agent-navigable requirements via MCP and companion skill" is a User Need: external agents, and the people who run them, navigating and safely changing requirements are customers being served.

## STPA Guidance (Safety & Reliability Analysis)

When guiding users through safety, reliability, or hazard analysis using STPA (Systems-Theoretic Process Analysis), enforce the **iterative STPA workflow across User Needs, Product Requirements, and Designs**:

1. **Optional Orthogonal Layer (Per-Project Toggle):**
   - STPA is an **optional orthogonal analysis layer** atop trace requirements, enabled or hidden via a **per-project toggle**. Projects with STPA toggled off remain completely uncluttered (no STPA tabs or mandatory fields), and toggling STPA off preserves all underlying STPA records and TraceLinks intact.
   - Maintain Losses, Hazards, Control Actions, Unsafe Control Actions (UCAs), Loss Scenarios, and safety Constraints in the STPA layer—never mix them into Business Needs, User Needs, or Product Requirements lists, and never duplicate STPA text inside PRDs. Link requirements, designs, implementations, and verification tests to STPA records by reference via TraceLinks.

2. **Pre-Design Losses & Hazards Derive from User Needs (`UN-`):**
   - **Losses** and **Hazards** (system states or conditions that can produce one or more Losses if left uncontrolled) derive from User Needs (`UN-`) and may be named before any Product Requirement or Design exists.
   - A Hazard is an analysis artifact, not a User Need or Product Requirement, and **safety Constraints do NOT live on Hazards**.
   - Link each Hazard to the Loss(es) it can cause and to the User Need(s) it derives from or informs. Flag any Hazard without a linked Control Action or design-level Constraint as an uncontrolled hazard gap.

3. **Product-Requirements-Level Control Actions (`PRD-`) & Stimuli (User-Initiated or Environmental):**
   - **Control Actions are first defined at the Product Requirements (`PRD-`) level** — the product defines *who does what* (e.g., driver steers via steering wheel to govern vehicle direction), stated in the affirmative and completely free of implementation architecture (saying nothing about e.g. fly-by-wire vs. rack-and-pinion).
   - **Control-Action stimuli may be user-initiated or environmental** (e.g., a human operator action, rain triggering windshield wipers, or a skid triggering stability control).
   - Product-level Control Actions trace up to the Product Requirement (`PRD-`) they belong to and link to the pre-design Hazard(s) they control.

4. **Four UCA Failure Modes Run Right After PRDs (Before Implementation Design):**
   - Immediately after Product Requirements and product-level Control Actions are defined—and **before** committing to an implementation Design—evaluate all four **Unsafe Control Action (UCA)** failure modes on each Control Action:
     1. `not_provided` — Not followed or not provided when needed (given the user-initiated or environmental stimulus)
     2. `provided_unsafely` — Followed or provided when unsafe
     3. `wrong_timing` — Followed too early, too late, or out of order
     4. `wrong_duration` — Applied too long or stopped too soon
   - Record UCAs under their parent Control Action and link each UCA to the specific Hazard(s) it can produce. **Never enumerate UCAs or negative failure modes as negative Product Requirements.**

5. **Iterative Design Pass, Subsystem Control Actions, and Design-Gated Constraints (`Constraint -> Control Action` & `Constraint -> Design -> PRD -> UN`):**
   - **Iterative Cycle (`Control Actions ↔ Design`):** STPA is an iterative feedback loop, not a strict one-way waterfall. Product-level Control Actions and their UCAs shape the initial Design (`SD-`, `UXD-`, or `CD-`, which traces up to `PRD-`), and committing to a specific Design architecture (e.g., choosing fly-by-wire vs. mechanical rack-and-pinion) reveals deeper **subsystem-level Control Actions** that are only visible after that design commitment.
   - **Deeper Design Pass:** Run a second STPA pass against the chosen Design (`SD-` / `UXD-` / `CD-`) to identify subsystem controllers and subsystem Control Actions, then walk the same four UCA failure modes (`not_provided`, `provided_unsafely`, `wrong_timing`, `wrong_duration`) and derive **Loss Scenarios** (causal pathways in the Design's control loop leading to a UCA, Hazard, and Loss).
   - **Design-Gated Safety Constraints:** An initial Design (`SD-`, `UXD-`, or `CD-`) **must be in place before safety Constraints can be finalized**. Every safety Constraint states an affirmative design-level obligation and **traces both to the Control Action(s) (and UCA/Loss Scenario) it enforces and up to the Design (`SD-`/`UXD-`/`CD-`), which in turn traces up to the Product Requirement (`PRD-`) and User Need (`UN-`)**.
   - **End-to-End Verification:** Implementations (`IMPL-`) and Verification Tests (`VER-` / `UT-`) trace to the Designs, Constraints, Control Actions, and Loss Scenarios they implement and verify, surfacing any uncontrolled Hazard, unaddressed Control Action, or unverified Constraint/Loss Scenario as a coverage gap.

6. **Free-Form Tagging (`STPA`):**
   - Attach free-form tags (`tags: ["STPA"]` via `ih_propose_requirement`, `ih_propose_edit`, or `ih_set_tags`) to STPA records, related requirements/designs/tests, and pending proposals so users and agents can filter requirements views, chain queries, search, and the Review queue (`tag=STPA`) to inspect and review related items together. All agent-authored STPA proposals require human Acceptance in the Review queue.

## Anti-patterns

- Echoing a secret or token anywhere
- Writing without reading neighbors, or adding duplicates
- Orphan PRDs (no parent UN). A TraceLink may name a still-pending requirement.
- Filing a Suggest TraceLink that has one pending-add end as its own Review item, showing a link between two pending adds as its own Review row, or treating a link between two existing requirements as already approved
- A requirement or revision body that explains background or a related capability instead of stating that requirement
- Asking for a `review` scope (**422**) or trying to Accept your own proposals
- Writing a living record from a Suggest credential, or from an Edit credential when the call says `mode: "suggest"` — that change waits in Review
- Editing a product that is not in `editDirectlyProductIds` and not covered by `edit` scope

## Changelog

- 1.0.0 (2026-10-13) — first release.
- 1.1.0 (2026-10-13) — renamed to "Ideation Hub bot"; REST-only description; added Design controls basics primer.
- 1.2.0 (2026-10-14) — TraceLinks write directly. No Review proposal. Hidden until both ends are canonical. A deleted or obsolete end hides the row; the row is kept.
- 1.3.0 (2026-10-13) — added "Working with the user (conversational workflow)" section.
- 1.3.1 (2026-10-13) — Profiles declare only the person, not their needs; User Needs state WHAT (objectives), not HOW; layered Profile/UN/PRD pattern on every create or update; recommend clearer structure instead of transcribing.
- 1.3.2 (2026-10-13) — "Bot credentials (guide the user)" step-by-step setup; step 6 PRD rubric wording fixed.
- 1.3.3 (2026-10-16) — rubric: a User Profile must name a role or type of person with a job to do, not a mechanism or single capability; such proposals are rejected and the capability proposed as a PRD under an existing Profile.
- 1.3.4 (2026-10-16) — rubric: every User Need must trace up to one or more User Profiles.
- 1.3.5 (2026-10-16) — rubric: Business Need vs User Need; BNs serve the owning business only, anything helping a customer is a UN traced to its Profile; misfiled items are re-proposed, not deleted.
- 1.3.6 (2026-10-16) — `POST /api/requirements/{id}/unaccept` moves an accepted requirement back to pending, keeps its Human Readable Durable ID, and can restore an earlier revision as a new revision. History and TraceLinks stay.
- 1.3.7 (2026-10-16) — reclassify: `POST /api/requirements/{id}/reclassify` moves an accepted or pending requirement to a new type, assigns a fresh human id, and keeps the old id as a resolvable alias. Direction-invalid TraceLinks are reported and kept.
- 1.3.8 (2026-10-16) — soft delete and restore: `POST /api/requirements/{id}/delete` and `POST /api/requirements/{id}/restore`. Deleted requirements hide from lists, Review, and traces; rows, human ids, and TraceLinks stay. Product owners, tenant admins, and edit_directly apply it; propose-scope agents file a delete proposal. Human ids are not reused. HTTP DELETE remains the no-hard-delete guard.
- 1.4.0 (2026-10-24) — MCP at `/api/mcp`. Setup link `https://www.ideationhub.me/settings/agents?setup=grokbot`. Read, Suggest (`propose`, review queue), and Edit (`editDirectlyProductIds` or `edit` scope writes the living record unless `mode` is suggest).
- 1.4.1 (2026-10-30) — §4 withdraw warning: `withdraw_proposal` on a pending-add requirement withdraws the whole requirement; withdraw only the proposal (or propose the replacement and ask the issuer to Reject the old one); PRD-038 example.
- 1.4.2 (2026-10-30) — `withdraw_proposal` takes a proposal id and closes only that proposal. A requirement id is rejected. `withdraw_requirement` is the explicit whole-requirement action. `reopen_requirement` restores a withdrawn pending requirement and its add proposal.
- 1.4.3 (2026-11-03) — `ih_link_requirements` and `ih_unlink_requirements` follow the suggest-versus-edit rule. Suggest files a `kind=link` proposal (`outcome: proposed`, `proposalId`). Edit applies it (`outcome: applied`, `link`). Read-only is 403.
- 1.4.4 (2026-11-08) — Partial TraceLink review, reconciling shared skill 1.3.10 and 1.3.11 into this 1.4 copy. Suggest add-links with exactly one pending-add end are bundled with that requirement's Accept, either direction, and the accept snapshot lists them as `approvedLinks`. Existing-to-existing links stay standalone Review items. A link between two pending adds still appears as its own Review row; that is a known gap, not a hidden row and not a completed rule. Direct edit, read-only denial, and link removal stay as in 1.4.3. New requirement and revision text stays scoped to that requirement's purpose. Does not replace 1.4.2 withdraw semantics.
- 1.4.5 (2026-11-08) — `ih_propose_edit` amends this credential's own pending add or edit in place. Same proposal id. A pending add stays in step with the unapproved draft. A pending edit leaves the accepted requirement unchanged until Accept. `proposalId` is required when more than one of yours is pending. Another author is rejected. Pending-only. Does not change 1.4.4 link bundling.
- 1.4.6 (2026-11-08) — a Suggest link between two pending adds stays pending, is hidden from standalone Review, and is applied with the last pending end. Accepting one end does not approve the other. `reopen_proposal` restores one withdrawn add-link proposal at version 1 without writing a trace. Removals are unchanged.
- 1.4.7 (2026-11-08) — the same suggest credential that created a pending add may call `ih_restore_draft_title`. The creator or the recorded withdrawing actor may call `reopen_proposal`. Editors keep their existing access. A foreign credential is refused. The snapshot checks, ids, proposal payload, statement, and traces stay as in 1.4.6.
- 1.4.8 (2026-11-09) — semantic duplicate checks + clarify ambiguity; check traces on every change and explain related changes with rationale before acting; no "As a X, I need" user-story form in User Needs, UN-097 example.
- 1.4.9 (2026-11-09) — reconciled with the published plugin copy (ideationhub/ideation-hub-cursor-plugin 1.4.0) so both copies match: kept its STPA Guidance section, Upward Traceability & Discovery Guidelines, Design/Implementation/Test record types and id prefixes, broader `realized_by` / `checked_by` definitions, MCP-header authentication, MCP tool parameter signatures, and MCP tool names in the conversational and propose workflows. No behavior rules changed from 1.4.8.

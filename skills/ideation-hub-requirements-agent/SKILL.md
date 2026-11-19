---
name: ideation-hub-requirements-agent
description: Connect coding agents to Ideation Hub (www.ideationhub.me) via the MCP server at /api/mcp or REST API to read, validate, trace, suggest, or directly edit Business Needs, User Profiles, User Needs, Product Requirements, Designs, Implementations, Tests, and TraceLinks, and to retrieve or structure requirements for an objective.
version: 1.6.0
---

# Ideation Hub — Requirements & Traceability Agent Skill

**Product:** `https://www.ideationhub.me` · **API:** `https://api.ideationhub.me` · **MCP:** `https://api.ideationhub.me/api/mcp`  
**Audience:** Any coding agent, Cursor agent, or Grok Bot working on a customer's Product in Ideation Hub. Tenant-agnostic: this file holds no secret, tenant id, or product id.

## Design controls basics

Ideation Hub is a requirements workspace built on **design controls**: a traceable chain from business intent to verified product behavior, as used in regulated (e.g. medical device) and ALM practice. Each record answers one question; links show how they connect.

- **Business Need** (`BN-NNN`) — a business objective. *Purpose:* why the business invests. *Good one:* one outcome the business wants, measurable where possible, free of solution detail.
- **User Profile** — who the product serves. *Purpose:* anchor needs to a real role. *Good one:* a clear `name` plus a `roleIntent` saying what that role is trying to get done. It declares only the actual person (who they are, their role and context), never their needs; those are User Needs. It is a role or type of person with a job to do, never one capability (see the Profile capability rule). The API identifies it by `UP-` id, UUID, or exact name.
- **User Need** (`UN-NNN`) — what a user must be able to do. *Purpose:* the user's view of value. *Good one:* "Ability to …", one capability, linked to every Profile it serves; WHAT the user wants to achieve, never HOW (that is the PRDs tracing up to it); it `addresses` the Business Needs it serves.
- **Product Requirement** (`PRD-NNN`) — WHAT the product must do. *Purpose:* a buildable, checkable commitment. *Good one:* concise, testable, no who/why or design (see the PRD rubric below); it `addresses` the User Needs and Business Needs it satisfies.
- **Design & Implementation** (`SD-NNN`, `UXD-NNN`, `CD-NNN`, `IMPL-NNN`) — System Design, UX Design (`mockupUrl`), Component Design, and Implementation (`repoPath`, `prUrl`, `commitSha`).
- **Verification & Validation Tests** (`VT-NNN`, `VER-NNN`, `UT-NNN`) — Validation Tests (check UNs), Verification Tests (check PRDs), and Unit Tests (`testPath`, `lastResult`: `pass` | `fail` | `not_run`).
- **Acceptance criteria / verification** — pass/fail checks live in Verification, Validation, and Unit Test records, never in a PRD body.

**Traceability.** Traces run from a child up to its parents, such as PRD → UN → BN. They let you prove coverage, spot gaps, and run impact analysis. What may trace to what is in [Traceability rules](#traceability-rules).

TraceLink types:

- `addresses` — child → parent (UN addresses BN; PRD addresses UN or BN).
- `refined_by` — a record is refined by a more detailed one.
- `realized_by` — a need is realized by a requirement, or a requirement or design by a downstream design or implementation.
- `checked_by` — a need, requirement, or implementation is checked (verified or validated) by another record, such as a test.

Walking **up** answers *why* this exists; walking **down** answers *how* it is delivered and checked. A **gap** (a record with no parent, a BN with no UN, a UN with no PRD, a PRD with no checks) is allowed. Flag it for the user.

Every requirement has a **Human Readable Durable ID** (`BN-`, `UN-`, `PRD-`, `SD-`, `UXD-`, `CD-`, `IMPL-`, `VT-`, `VER-`, `UT-` + number, sequenced per type inside a Product) plus a UUID and a revision `@n`. Human Readable Durable IDs never get renumbered or reused.

## Traceability rules

Every trace is many-to-many, with zero or more on each side, and existing links are always preserved.

Allowed upward traces (child → parent). A Business Objective is also called a Business Need.

- Product Requirement → Business Objective and User Need
- User Need → User Profile
- System Design → Product Requirement
- UX Design → Product Requirement
- Component Design → System Design
- Implementation → Component Design, UX Design and System Design
- Validation Test → Product Requirement, UX Design, System Design, Component Design and Implementation
- Verification Test → User Need and Business Objective
- Unit Test → Implementation and Component Design

**Hard rule:** a Product Requirement never links directly to a User Profile. It reaches a profile only through a User Need.

**Profile capability rule:** a proposed Profile that is really one capability (e.g. "issues scoped agent credentials") is not a Profile. Propose that capability as a Product Requirement that traces to a User Need of an existing Profile, not to the Profile itself.

A record with no upward trace is allowed; flag it as a coverage gap. A one-to-one trace needs an explicit stated reason.


## Reference files (load only when the task needs them)

- `references/retrieval.md` — retrieve requirements for an objective: search, vertical slice, horizontal view, or full export; how to read IDs and links; token budget.
- `references/document-structure.md` — structure a document: point first, non-overlapping sections in task order; derive and order PRD sections; write section summaries.
- `references/rest-api.md` — REST routes, OAuth token, and exact lifecycle behavior (edit, link, reclassify, un-accept, soft delete, withdraw, reopen).
- `references/stpa.md` — STPA safety and hazard analysis.
- `references/changelog.md` — version history.

## Working with the user (conversational workflow)

1. **Have a conversation with the end user or parse a document from them.**
2. **Be interactive and ask for disambiguation of anything you hear.** Pair open questions with suggested interpretations ("Did you mean A or B, or something else?"); never guess.
3. **Disseminate all information from the conversation with the end user** — break everything the user said into candidate requirement elements (BN, Profile, UN, PRD, acceptance criteria, designs, tests), losing nothing.
4. **Retrieve what already exists for the objective.** Start with the smallest read that answers it: `ih_search` for one thing, `ih_get_neighbors` for one area, `ih_get_chain` only for a whole-product view. See `references/retrieval.md`.
5. **Have a dialogue with the user about what currently exists in the requirements** — matches, overlaps, gaps.
6. **Determine how best to update those requirements or add to them, keeping the design control structure in place:** TraceLinks up to UNs/business needs, the PRD rubric (testable product behavior, no implementation), and propose-only via Review (§4) unless direct edit is explicitly granted. Apply the layered pattern (see Writing rules): Profile = the person, UN = the WHAT, PRD = the HOW as testable product behavior.
7. **Recommend, don't just transcribe.** If the user is misguided (e.g. needs in a Profile, a HOW in a User Need), still recommend how to define the requirements clearly: explain the better structure and offer a rewritten version.

## Upward Traceability & Discovery Guidelines (Required When Authoring Requirements)

1. **Trace each PRD up to every User Need and Business Need it satisfies (`PRD --addresses--> UN` or `BN`).** See [Traceability rules](#traceability-rules). Link it via `parentId` on create and/or `ih_link_requirements` (`sourceId: "<PRD-ID>", type: "addresses", targetId: "<UN-ID>"`).
2. **Search existing User Needs whenever defining a new requirement.** Before creating or updating a requirement, search all existing User Needs in the Product (`ih_search`, or `ih_get_chain` for the whole set) and consider which existing User Needs that new requirement satisfies.
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

`POST /api/oauth/token` returns a Bearer token pinned to tenant and Product. Request and error details: `references/rest-api.md`.

- Permissions: `read`, `suggest` (stored as `propose`), and direct edit via `editDirectlyProductIds` or an `edit` scope. Missing permission → **403**. `review` is not a scope.

## 3. MCP (preferred)

Streamable HTTP JSON-RPC at `POST https://api.ideationhub.me/api/mcp` (also `/api/v0/mcp` and `/mcp`). No SSE. `GET` returns server metadata. Headers: `X-IH-Client-Id`, `X-IH-Client-Secret`. A bearer token from §2 also works.

`initialize` returns protocol `2025-03-26` (or `2024-11-05` when requested), server `ideation-hub-mcp`. `tools/list` and `tools/call` require an agent credential. Omit `productId` to use the credential's current product. A product outside the grant is **403**. `ih_list_products` returns only granted products.

| Tool | Permission | What it does |
| --- | --- | --- |
| `ih_list_products` | read | `{folderId?}`. Granted products (`directEdit` per product) and visible folders |
| `ih_get_chain` | read | `{productId?, includePending?, includeObsolete?, tag?, tags?}`. Profiles, requirements, and links. `includePending` keeps unapproved rows. `includeObsolete` defaults to true; pass false for living records. Optional `tag` or `tags` (AND, case-insensitive). Large; see `references/retrieval.md` |
| `ih_search` | read | `{query, types?, status?, includePending?, tag?, tags?, limit?, cursor?, productId?}`. Hybrid meaning and keyword search across requirements, profiles, PRD sections, verification specs, evidence, and pending proposals. Returns each match with its best passage, `score`, and `matchReason`. An exact human id ranks first |
| `ih_search_requirements` | read | `{query, types?, status?, productId?, limit?, tag?, tags?}`. Keyword listing by human id, title, and body; not meaning-based. Tag filters are AND |
| `ih_get_requirement` | read | `{id, productId?}`. Human id (`PRD-001`) or UUID, revision, design fields, tags, inbound and outbound links, `pendingProposal` (null or the one pending add or edit, including `payload` and `canAmend`), `pendingProposalIds` |
| `ih_get_neighbors` | read | `{id, direction?: up\|down\|both, depth?, includePending?, includeObsolete?, productId?}`. `depth` defaults to 1. Pending neighbors are kept and obsolete ones dropped by default |
| `ih_validate_against_trace` | read | `{id?, proposedPayload?, productId?}`. Duplicate title, missing profile or parent, orphan. Does not write |
| `ih_list_review_queue` | read | `{status?, source?: all\|agent\|human\|mine, type?, productId?, tag?, tags?}`. Each item's tags merge the payload and the target requirement |
| `ih_propose_requirement` | suggest or edit | `{type, statement, title?, profileNameOrId?, parentId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, tags?, mode?, productId?}`. Add BN/UN/PRD/SD/UXD/CD/IMPL/VT/VER/UT. See the write rule below |
| `ih_propose_edit` | suggest or edit | `{id, title?, statement?, expectedRevision?, proposalId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, tags?, addLinks?, removeLinks?, addProfileIds?, removeProfileIds?, mode?, productId?}`. Revise. Amends the one pending add or edit on that requirement in place (`amended: true`, same `proposalId`), whoever filed it (§4 step 7). `addLinks` and `removeLinks` are arrays of `{type, targetId \| targetProfileId}`; on a User Need, `addProfileIds` and `removeProfileIds` are shorthand. Link changes apply with the edit at Accept; a conflict then is **409** `link_conflict` or `duplicate_link`. **409** `pending_proposal_exists` or `ambiguous_proposal`: amend or pass that `proposalId`. `expectedRevision` mismatch is **409**. A no-op is **422** |
| `ih_propose_bulk_push` | suggest or edit | `{items, links?, dryRun?, mode?, productId?}`. `dryRun` does not persist or consume human ids |
| `ih_propose_profile` | suggest or edit | `{action: add\|edit, id?, name, roleIntent, mode?, productId?}` |
| `ih_create_requirement` / `ih_update_requirement` | edit only | Direct aliases. Optional `tags`. Suggest-only is **403** |
| `ih_set_tags` | edit only | `{id, tags, productId?}`. Replaces tags. Does not create a revision. Suggest-only is **403** |
| `ih_link_requirements` | suggest or edit | `{sourceId, type: addresses\|realized_by\|checked_by\|refined_by, targetId?, targetProfileId?, productId?}`. Add a TraceLink. Pass exactly one of `targetId` or `targetProfileId`; a User Need to User Profile link uses type `addresses` and `targetProfileId` only. Outcomes: `applied` (edit), `bundled` with a pending add (`bundledWith`), or `proposed` (`proposalId`). See TraceLink review in §4. **422** `link_pair_not_allowed`, `profile_not_in_product`, or `link_target`. **409** `duplicate_link`. Proposing the same pending link again returns the existing proposal |
| `ih_unlink_requirements` | suggest or edit | `{linkId?, sourceId?, type?, targetId?, targetProfileId?, productId?}`. Soft-remove by `linkId` or source + type + target. To remove a User Need to User Profile link, pass `targetProfileId` instead of `targetId`. Same applied-versus-proposed split. Reject or withdraw leaves the link |
| `ih_lifecycle_action` | suggest or edit | `{id, action, targetType?, reason?, mode?, productId?}`. Actions: `withdraw_proposal` (proposal id only), `withdraw_requirement`, `reopen_proposal` (withdrawn add-link at version 1; creator, withdrawing actor, or editor), `reopen_requirement` (edit), `obsolete`, `delete_draft`, `reclassify` |
| `ih_restore_draft_title` | suggest | Title-only repair of the caller's pending add. Full snapshot required. Does not change the proposal |
| `ih_record_development_phase_event` | delivery:write | `{requirement, phase, sourceId?, requester?, evidenceUrl?, occurredAt?, productId?}`. Append one development-phase event. A direct write, not a proposal: events are not requirement content |

**Write rule.** If the credential has direct edit on the target product (`editDirectlyProductIds` or `edit` scope) and the call does not pass `mode: "suggest"` or `"propose"`, the change is applied to the living record and audited with the agent client id and the issuer user id, except that a direct edit while a proposal is pending is **409** `pending_proposal_exists`; amend that proposal instead. Otherwise it is a pending proposal (`proposalId`, `reservedHumanId`, `status: "pending"`, `reviewUrl`) on the issuer's Review Queue. A TraceLink with exactly one pending-add end is the exception in §4: it is stored with that requirement instead of a standalone proposal. Pass `mode: "suggest"` to queue even when direct edit is allowed. Two pending adds stay one pending proposal and are hidden from standalone Review until both ends are approved. `mode: "edit"` without direct edit is **403**. Agents cannot Accept or Reject (**403**).

Design fields on UXD / IMPL / VT / VER / UT: `mockupUrl`, `repoPath`, `prUrl`, `commitSha`, `testPath`, `lastResult` (`pass`, `fail`, `not_run`; `passed` and `failed` are accepted). A User Need takes `profileNameOrId`. A Product Requirement takes `parentId` or `unId` and `addresses` that User Need.

Tags are a free-form list (`tags: string[]`). Each tag is 1–32 characters from letters, numbers, spaces, and `_-./`, at most 20, trimmed and de-duplicated without regard to case. Omit `tags` to leave them unchanged. `[]` clears them. Responses always include `tags` (an empty list when there are none). `ih_set_tags` and `PUT /api/requirements/{id}/tags` replace the list without a new revision. Review items also show tags stored on the target requirement, so a pending proposal can be filtered by a tag that was seeded on that row.

REST mirrors these tools and follows the same read / suggest / edit split. Routes: `references/rest-api.md`.

## 4. Propose a change (write)

A **Suggest** credential (`read` + `propose`, no direct edit on that product) files a **proposal**. It lands in **Review**; the living record stays `unapproved` until the credential issuer Accepts. An **Edit** credential (`editDirectlyProductIds` for that product, or `edit` scope) writes the living record unless the call passes `mode: "suggest"` or `"propose"`. Agents cannot Accept or Reject (**403**).

**TraceLink review.**

- **One end is a pending add:** a Suggest link is stored with that requirement (`outcome: "bundled"`, `bundledWith` is its Human Readable Durable ID), either direction. No standalone Review item; it shows on that requirement's Review card. Accept keeps it and records `approvedLinks`. Reject or withdraw of the add hides it. A requirement with only a pending edit counts as existing.
- **Both ends exist:** the link is its own Review item (`outcome: "proposed"`, `proposalId`). Accept applies the add or the soft-remove. Reject and withdraw leave the trace unchanged.
- **Both ends are pending adds:** one pending proposal, not bundled and not shown in standalone Review while either end is pending. Accepting one end does not approve the other. Accepting the link early is **409** `ends_pending`. The trace is written when the last pending end is accepted.
- Direct edit writes the row (`outcome: "applied"`). Read-only is **403**. Removing a link: Suggest files a removal proposal; direct edit soft-removes it.

Soft delete is a proposal for Suggest, and applies immediately when that credential has direct edit.

Workflow:

1. **Read neighbors first.** Pull the chain (`ih_get_chain`) or neighbors (`ih_get_neighbors`) for the parent you will attach to.
2. **Check duplicates, conflicts, overlap, and related requirements before writing.** Search existing requirements and pending proposals by meaning as well as title and statement, including their profiles, scope, status, and traces. Compare against same-type siblings under the intended parent and same-type records product-wide, and state the one-line scope boundary before proposing (see Scope boundary and overlap). Prefer updating an existing requirement only for a full duplicate; if overlap is partial, ask. For every proposed new or updated requirement, evaluate other related requirements and all appropriate upward trace targets; link only where the proposed capability genuinely satisfies the need, not merely because wording is similar. Preserve distinctions such as tenant-wide versus product-specific authority.
   - **Clarify ambiguous definitions or structure before any write.** If duplicate, conflicting, overlapping, or related requirements leave ambiguity about which record to create or update, whether to combine or separate capabilities, or which traces are appropriate, explain the relevant records and recommend a structure, then ask the user to resolve that ambiguity. Do not silently merge, broaden scope, withdraw, or invent relationships. Clear, non-conflicting matches do not require another confirmation. A second `add` of an existing key conflicts.
3. **Consider traces whenever modifying a requirement.** Before any edit, withdrawal, obsolete, reclassify, or link change, read the requirement's inbound and outbound traces (`ih_get_requirement`, `ih_get_neighbors`). Check whether each existing trace still holds after the change, whether any should be added, removed, or redirected, and whether linked requirements need their own updates. Where you are not certain a trace or related change is right, ask the user for guidance rather than guessing. After a trim, split, or merge, re-check that no obligation is unowned or double-owned and that traces still hold.
4. **Explain planned related changes and ask before acting on recommendations.** Before writing, tell the user the main change and every related change you plan (trace additions or removals, edits to linked requirements, withdrawals), with a one-line rationale for each, and ask any open questions. Proceed only with what the user confirms; changes the user explicitly asked for need no second confirmation, but related changes you recommend do.
5. **Editor pass before proposing (required).** Clear, readable requirements are a primary goal. Reread each title, then each body, as a careful human editor would for a reader new to the product: Is the subject a real thing in the product? Is the verb accurate? Could any word or phrase be read two ways? Is any sentence too long, or does it bury its point behind a list? Would the reader stumble or need context? Rewrite until it reads naturally and cannot be misread; titles get the most care.
6. **Explain-back check before proposing.** Launch a fresh subagent and give it only the requirement's title and body, with no other context. The subagent explains in plain words what the Product must do and for whom. Judge whether that matches what you meant. Check the title and the body for clarity, for making sense on their own, and for consistency with each other. Rewrite anything the reader misread or found ambiguous (titles most of all), then repeat with a new fresh reader, up to 3 rounds. If you cannot launch a subagent, do the editor pass in step 5 instead.
7. **Amend a pending proposal instead of filing another.** Before proposing a change to a requirement, read it and check `pendingProposal`. If one exists, read its `payload` and amend it with `ih_propose_edit` plus `proposalId`. Do not edit the requirement directly or file a second proposal. On **409** `pending_proposal_exists`, amend that `proposalId`. Anyone with propose permission can amend any pending proposal. After accept, reject, or withdraw, a new proposal can be filed.
8. **Validate and propose with a parent.** Use `ih_validate_against_trace`, or `ih_propose_bulk_push` with `dryRun: true` when staging several changes, before writing.
9. **File a link with the matching call.** To add or remove only a link, use `ih_link_requirements` or `ih_unlink_requirements`. To change links as part of an edit, pass `addLinks` and `removeLinks` on `ih_propose_edit` (or `addProfileIds` and `removeProfileIds` on a User Need). A User Need traces to a User Profile with type `addresses` and `targetProfileId` set to the profile human id (for example `UP-004`); do not also pass `targetId`. Suggest mode files a proposal; Accept applies only the named link and keeps the others. Keep the [Traceability rules](#traceability-rules).

Exact REST calls and lifecycle behavior (reclassify, un-accept, soft delete, restore, reopen) are in `references/rest-api.md`. These rules always apply:

- Failed propose → **422** (unknown `type` → **400**); nothing is stored. A pending UN or Profile is not a valid parent; wait for Accept, then propose the child.
- Requirements are never hard-deleted. Human Readable Durable IDs are never renumbered or reused.
- **No-op edits:** do not propose an edit whose title and statement match the current revision.
- Never re-propose to dodge a Reject; ask the human.
- **Withdraw one proposal** with `ih_lifecycle_action` `action: "withdraw_proposal"` and the **proposal id**. A requirement id is **422** `requirement_id`. Withdraw a whole pending requirement (`withdraw_requirement`) only when the user asks for exactly that.

## 5. Report back to the human

After proposing, tell the issuer in one short message:

- Each proposal `id`, `kind` (add/edit/obsolete/…), and the target Human Readable Durable ID or title. For a TraceLink, report the link id when the result is `bundled` or `applied`, or the `proposalId` when it is a standalone review between two existing requirements, and say which of those it is.
- Where to decide: **Review changes** on www (`https://www.ideationhub.me`, avatar → *Agent proposals*, or the Product's Review tab).
- Anything rejected with **422** and why. Never include secrets or tokens.

## Writing rules (PRD rubric)

- **Limit each body to that requirement's purpose.** On a new requirement or a revision, include only the detail that states that requirement. Omit background and explanatory subject detail that is not needed to state it. A separate capability belongs in its own traced requirement.
  - Example of this rule, not a change to a living record: a requirement whose purpose is an optional per-project STPA layer states that per-project on/off capability. It does not enumerate what the STPA layer contains or explain STPA.
- **Plain, unambiguous sentences (every type).** A newcomer should understand each title and body on first read. Name a clear subject that exists in the product, with an accurate verb (a permission declares; it does not decide). No jargon, coined phrases, or internal shorthand (e.g. "no silent living writes"). Do not use words with more than one meaning. When drafting or revising, reread the text, check that nothing can be read more than one way, and rewrite any ambiguous term so it has the one intended meaning. Keep sentences short and state the point first; put any list after it instead of packing many items into one sentence. Run the editor pass in §4 step 5. Stay concise.
- **Titles never name a User Profile or role** (for example "Agents …", "Product Owner can …", "Developers …"). The trace from the Product Requirement up to its User Need, and from there to the Profile, already carries who it is for. Write the title as the capability itself.
- **Titles state what the Product does**, starting with an action verb ("Show …", "Record …", "Search …"). Do not use a bare noun phrase naming a concept or metric, where words like "lead time" could read as a noun or a verb.
  - Avoid: "Lead time and its trend for each Product Requirement"
  - Prefer: "Show lead time and its trend for each Product Requirement"
- A PRD states only **WHAT the product must do**, concisely. A checker can pass/fail it without the linked UN.
- No who / why / role / purpose / outcome — the TraceLinks up to the UN carry that.
- No component counts, UI surfaces (pages, dialogs, controls), implementation shape, or design/engineering decisions.
- No acceptance criteria in a PRD body. Pass/fail checks belong in Verification, Validation, or Unit Test records that trace to the PRD.
- No IDs in prose and no "extends PRD-…" in statements. Express relationships as TraceLinks.
- User Needs read "Ability to …" and state the user's objective (WHAT), not HOW it is delivered; PRDs that trace up to the UN carry the HOW.
- **No user-story form in User Needs.** Never write "As a <role>, I need …" or otherwise restate who the user is; the User Profile trace already carries the role, so it is redundant. State only the objective, simply, without mechanisms such as credentials, permission models, CRUD lists, or approval workflow (those belong in the PRDs beneath).
  - *Example (UN-097, 2026-11-09):* Bad: "As a Product Owner enabling agents, I need an agent with Product-scoped credentials and RBAC to Create, Read, Update, and Delete requirements on that Product via MCP, with Create/Update/Delete as proposals only." Good: "Create an account for agents to read and propose requirement changes to a product I own."
- A User Profile declares only the person (who, role, context), never their needs or one capability; see the Profile capability rule. Link each User Need to every Profile it serves. Reject a Profile proposal that fails this, and give the reason.
- **Layered pattern — apply on every create or update:** User Profiles declare only the person; User Needs declare only the WHAT (the user's objectives); Product Requirements declare the HOW, each tracing up to the User Needs it satisfies (see [Traceability rules](#traceability-rules)). The PRD's "how" is the product capability that satisfies the need, stated as testable product behavior, never implementation or design.

**Bad:** "MCP server at /api/mcp with existing agent surface plus three tools." (count + implementation)  
**Good:** "Search requirement titles and bodies within one Product." (one behavior a test can check, no implementation)

## Scope boundary and overlap (rubric)

Apply before every add or edit. Breadth is fine; overlapping another same-type requirement is not.

- **Boundary.** One line: this owns [capability] under [conditions]; it does not own [adjacent]. If you cannot name the outside, ask.
- **One coherent feature.** Fields and conditions under that feature are fine. Two separately testable features are not. One owner per obligation; do not copy it into another body — trace instead.
- **Overlap.** Same-type records both claiming the same obligation. Not overlap: parent vs child, a TraceLink, or the same words for a different actor or scope.
- **Fix.** Narrow, merge, split, or move the shared behavior to one owner and trace. Children of a parent partition its scope (flag gaps). A parent may be broader than any one child.
- **Ask** when overlap is partial, more than one fix is reasonable, the boundary is unclear, or traces look wrong. Never silently merge, broaden, or withdraw.

## Business Need vs User Need (rubric)

- A **Business Need** is what the business that owns the product must accomplish for itself: revenue, cost, compliance, strategy, or operations. It is not for the sake of the customer.
- Anything meant to help a customer or user is a **User Need**, linked to the relevant customer User Profiles.

Pass/fail checks when filing or reviewing:

1. **Who benefits directly?** The owning business → Business Need. A customer or user → User Need.
2. **Can you name the customer Profile it serves?** If yes, it is a User Need.
3. A Business Need statement contains no customer benefit or outcome.
4. Reviewers flag any misfiled item and re-propose it as the right type; never delete it.

Examples:

- BN-002 "Delivery speed is visible and measurable" is misfiled. As a User Need (Profile: Product Owner): "Ability to see how long my products take to build."
- BN-038 "Agent-navigable requirements via MCP and companion skill" is a User Need: external agents, and the people who run them, navigating and safely changing requirements are customers being served.

## STPA (safety analysis)

Safety, reliability, or hazard analysis follows an iterative STPA workflow kept in its own optional layer, never mixed into requirement text. Load `references/stpa.md` before doing any STPA work.

## Anti-patterns

- Echoing a secret or token anywhere
- Writing without reading neighbors, or adding duplicates
- Breaking a [Traceability rule](#traceability-rules)
- Leaving a record with no parent unflagged. A TraceLink may name a still-pending requirement.
- Filing a Suggest TraceLink that has one pending-add end as its own Review item, showing a link between two pending adds as its own Review row, or treating a link between two existing requirements as already approved
- A requirement or revision body that explains background or a related capability instead of stating that requirement
- Asking for a `review` scope (**422**) or trying to Accept your own proposals
- Writing a living record from a Suggest credential, or from an Edit credential when the call says `mode: "suggest"` — that change waits in Review
- Editing a product that is not in `editDirectlyProductIds` and not covered by `edit` scope

## Changelog

Current: 1.6.0 (2026-11-19) — progressive disclosure: retrieval and document-structure references; REST, STPA, and history moved to `references/`; duplicate guidance merged; no acceptance criteria in PRD bodies. Full history: `references/changelog.md`.

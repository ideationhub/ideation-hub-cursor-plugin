# MCP tools

Canonical catalog of the Ideation Hub MCP server. Tool arguments, permissions, and the suggest-versus-edit rule live here. The requirements plugin says when to author a record. The developer plugin says when to design, implement, or assess. Neither plugin repeats this catalog.

## Endpoint and auth

Streamable HTTP JSON-RPC at `POST https://api.ideationhub.me/api/mcp` (also `/api/v0/mcp` and `/mcp`). No SSE. `GET` returns server metadata.

Preferred headers: `X-IH-Client-Id: ${IH_CLIENT_ID}` and `X-IH-Client-Secret: ${IH_CLIENT_SECRET}`. The plugin `mcp.json` sends those headers. A bearer token from `POST /api/oauth/token` also works. Token request and errors: `rest-api.md`.

`initialize` returns protocol `2025-03-26` (or `2024-11-05` when requested), server `ideation-hub-mcp`. `tools/list` and `tools/call` require an agent credential. Omit `productId` to use the credential's current product. A product outside the grant is **403**. `ih_list_products` returns only granted products.

Never paste, echo, log, or print the client secret or an access token.

## Permissions

| Permission | Effect |
| --- | --- |
| `read` | Read and search. |
| Suggest (`propose`) | Writes land in the issuer's Review queue until a human Accepts. `suggest` and `propose` are the same permission. |
| Edit (`editDirectlyProductIds` or an `edit` scope) | A write without `mode: "suggest"` or `"propose"` changes the living record. Pass `mode: "suggest"` to queue a proposal anyway. |
| `delivery:write` | Append a development-phase event. Direct. Not a proposal. |
| `status:write` | Record a confirmation review or a status assessment. Direct. Never edits a title, a body, or a trace link. |

`review` is not a scope. Agents cannot Accept or Reject (**403**). Missing permission is **403**. Do not work around a 403 with another tool or credential.

**Write rule.** If the credential has direct edit on the target product and the call does not pass `mode: "suggest"` or `"propose"`, the change is applied to the living record and audited with the agent client id and the issuer user id, except that a direct edit while a proposal is pending is **409** `pending_proposal_exists`. Amend that proposal instead. Otherwise the call is a pending proposal (`proposalId`, `reservedHumanId`, `status: "pending"`, `reviewUrl`) on the issuer's Review queue. A trace link with exactly one pending-add end is stored with that requirement instead of a standalone proposal (`outcome: "bundled"`). Both ends existing: `outcome: "proposed"`. Direct edit: `outcome: "applied"`. Two pending adds stay one pending proposal and are hidden from standalone Review until both ends are approved. `mode: "edit"` without direct edit is **403**.

The developer plugin always passes `mode: "suggest"` on design and implementation writes unless the user, in that conversation, explicitly asks for a direct write. That is a plugin rule on top of this mechanism.

### Trace link review

- **One end is a pending add:** a Suggest link is stored with that requirement (`outcome: "bundled"`, `bundledWith` is its Human Readable Durable ID), either direction. No standalone Review item. It shows on that requirement's Review card. Accept keeps it and records `approvedLinks`. Reject or withdraw of the add hides it. A requirement with only a pending edit counts as existing.
- **Both ends exist:** the link is its own Review item (`outcome: "proposed"`, `proposalId`). Accept applies the add or the soft-remove. Reject and withdraw leave the trace unchanged.
- **Both ends are pending adds:** one pending proposal, not bundled and not shown in standalone Review while either end is pending. Accepting one end does not approve the other. Accepting the link early is **409** `ends_pending`. The trace is written when the last pending end is accepted.
- Direct edit writes the row (`outcome: "applied"`). Read-only is **403**. Removing a link: Suggest files a removal proposal; direct edit soft-removes it.

Soft delete is a proposal for Suggest, and applies immediately when that credential has direct edit.

## Tools

| Tool | Permission | What it does |
| --- | --- | --- |
| `ih_list_products` | read | `{folderId?}`. Granted products (`directEdit` per product) and visible folders |
| `ih_get_chain` | read | `{productId?, includePending?, includeObsolete?, tag?, tags?}`. Profiles, requirements, and links. `includePending` keeps unapproved rows. `includeObsolete` defaults to true; pass false for living records. Optional `tag` or `tags` (AND, case-insensitive) |
| `ih_search` | read | `{query, types?, status?, includePending?, tag?, tags?, limit?, cursor?, productId?}`. Hybrid meaning and keyword search across requirements, profiles, PRD sections, verification specs, evidence, and pending proposals. Returns each match with its best passage, `score`, and `matchReason`. An exact human id ranks first |
| `ih_search_requirements` | read | `{query, types?, status?, productId?, limit?, tag?, tags?}`. Keyword listing by human id, title, and body. Not meaning-based. Tag filters are AND |
| `ih_get_requirement` | read | `{id, productId?}`. Human id (`PRD-001`) or UUID, revision, design fields, tags, inbound and outbound links, `pendingProposal` (null or the one pending add or edit, including `payload` and `canAmend`), `pendingProposalIds` |
| `ih_get_neighbors` | read | `{id, direction?: up\|down\|both, depth?, includePending?, includeObsolete?, productId?}`. `depth` defaults to 1. Pending neighbors are kept and obsolete ones dropped unless you pass otherwise |
| `ih_validate_against_trace` | read | `{id?, proposedPayload?, productId?}`. Duplicate title, missing profile or parent, orphan. Does not write |
| `ih_list_review_queue` | read | `{status?, source?: all\|agent\|human\|mine, type?, productId?, tag?, tags?}`. Each item's tags merge the payload and the target requirement |
| `ih_list_sections` | read | PRD sections in order, with summaries |
| `ih_propose_requirement` | suggest or edit | `{type, statement, title?, profileNameOrId?, parentId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, tags?, mode?, productId?}`. Add BN/UN/PRD/PRN/SD/UXD/CD/IMPL/VT/VER/UT |
| `ih_propose_edit` | suggest or edit | `{id, title?, statement?, expectedRevision?, proposalId?, mockupUrl?, repoPath?, prUrl?, commitSha?, testPath?, lastResult?, tags?, addLinks?, removeLinks?, addProfileIds?, removeProfileIds?, mode?, productId?}`. Revise. Amends the one pending add or edit on that requirement in place (`amended: true`, same `proposalId`). `addLinks` and `removeLinks` are arrays of `{type, targetId \| targetProfileId}`. On a User Need, `addProfileIds` and `removeProfileIds` are shorthand. Link changes apply with the edit at Accept. A conflict then is **409** `link_conflict` or `duplicate_link`. **409** `pending_proposal_exists` or `ambiguous_proposal`: amend or pass that `proposalId`. `expectedRevision` mismatch is **409**. A no-op is **422** |
| `ih_propose_bulk_push` | suggest or edit | `{items, links?, dryRun?, mode?, productId?}`. `dryRun` does not persist or consume human ids. It checks types, required fields, duplicate titles, and that parents and link ends exist. It does not check link direction |
| `ih_propose_profile` | suggest or edit | `{action: add\|edit, id?, name, roleIntent, mode?, productId?}` |
| `ih_create_requirement` / `ih_update_requirement` | edit only | Direct aliases. Optional `tags`. Suggest-only is **403** |
| `ih_set_tags` | edit only | `{id, tags, productId?}`. Replaces tags. Does not create a revision. Suggest-only is **403** |
| `ih_link_requirements` | suggest or edit | `{sourceId, type: addresses\|respects\|realized_by\|checked_by\|refined_by, targetId?, targetProfileId?, productId?}`. Pass exactly one of `targetId` or `targetProfileId`. A User Need to User Profile link uses type `addresses` and `targetProfileId` only. `respects` runs from a requirement to a principle: the source is the requirement and the target is the principle. The inverse is shown as Respected by. Outcomes: `applied`, `bundled` (`bundledWith`), or `proposed` (`proposalId`). **422** `link_pair_not_allowed`, `profile_not_in_product`, or `link_target`. **409** `duplicate_link`. Proposing the same pending link again returns the existing proposal |
| `ih_unlink_requirements` | suggest or edit | `{linkId?, sourceId?, type?, targetId?, targetProfileId?, productId?}`. Soft-remove by `linkId` or source + type + target. To remove a User Need to User Profile link, pass `targetProfileId` instead of `targetId`. Reject or withdraw leaves the link |
| `ih_lifecycle_action` | suggest or edit | `{id, action, targetType?, reason?, mode?, productId?}`. Actions: `withdraw_proposal` (proposal id only), `withdraw_requirement`, `reopen_proposal` (withdrawn add-link at version 1), `reopen_requirement` (edit), `obsolete`, `delete_draft`, `reclassify` |
| `ih_restore_draft_title` | suggest | Title-only repair of the caller's pending add. Does not change the proposal |
| `ih_create_section` | suggest or edit | Add a PRD section |
| `ih_rename_section` | suggest or edit | Rename or nest a PRD section |
| `ih_delete_section` | suggest or edit | Delete a PRD section. Requirements in it become unassigned |
| `ih_reorder_sections` | suggest or edit | Reorder all PRD sections |
| `ih_set_section_summary` | suggest or edit | Set a PRD section summary |
| `ih_assign_sections` | suggest or edit | Place many PRDs into sections in one change |
| `ih_list_applicable_principles` | read | `{productId?}`. Principles that apply to this product. See below |
| `ih_propose_principle_adoption` | suggest or edit | `{principleId, action: adopt\|stop, mode?, productId?}`. Adopt or stop on this product. See below |
| `ih_list_principle_conflicts` | read | `{unresolved?, productId?}`. Conflicts for this product and the tenant. See below |
| `ih_propose_principle_conflict` | suggest or edit | `{principleA, principleB, priorityPrincipleId?, reason?, mode?, productId?}`. See below |
| `ih_record_development_phase_event` | delivery:write | `{requirement, phase, sourceId?, requester?, evidenceUrl?, occurredAt?, productId?}`. Append one development-phase event. Direct, not a proposal. The developer plugin records `phase: "design_ready"` at handoff |
| `ih_get_revision_status` | read | `{id, productId?}`. Per revision: derived completeness and coverage, the latest assessment or null, and `differs`, `completenessDiffers`, `coverageDiffers`, `completenessComment`, `coverageComment`. REST: `GET /api/requirements/{id}/revision-status` |
| `ih_record_confirmation_review` | status:write | See below. Direct. Do not pass `mode` |
| `ih_record_status_assessment` | status:write | See below. Direct. Do not pass `mode` |
| `ih_subscribe_requirement_status` | read | `{id, revisionAccepted?, deploymentCompleted?, statusChanged?, productId?}`. Ask to be notified about one Product Requirement. Sending the same subscription again updates the flags. Notices stay in an inbox. There is no outbound webhook |
| `ih_list_requirement_notifications` | read | Inbox for this credential, newest first |

Section tools are listed with the purpose the product documents. If `tools/list` shows a more specific argument schema for a section tool, use that schema. Do not invent arguments.

How to retrieve a small slice versus a whole product is the requirements plugin's `references/retrieval.md`. How to judge implementation and test coverage is the developer plugin's `references/assessment.md`. Those files use the tools above. They do not redefine them.

## Principles

Tenant-wide adoption is done by a human tenant admin in the web app, not through MCP. `ih_propose_principle_adoption` adopts a principle on the granted product only.

### Applicable principles

`ih_list_applicable_principles`. Read.

| Argument | Value |
| --- | --- |
| `productId` | Granted product |

Principles that apply to this product. Read. Each entry is an active adoption: `principleId`, `humanId`, `title`, `statement`, `revision`, `sourceProductId`, `sourceProductName`, `source` (`product` or `tenant`), `adoptionId`, and `adoptedAt`. `source` `product` is the product that recorded the principle. `source` `tenant` means the tenant adopted it, and that entry is included for every product in the tenant. A principle adopted at both scopes is listed once, with `source` `tenant`. Title, statement, revision, and source product name are returned even when this credential cannot open the product that recorded the principle. Ended adoptions are omitted.

### Product adoption

`ih_propose_principle_adoption`. Suggest or edit.

| Argument | Value |
| --- | --- |
| `principleId` | Required. Principle human id or uuid recorded in this product |
| `action` | Required. `adopt` or `stop` |
| `mode` | `suggest`, `propose`, or `edit`. Defaults to `suggest` |
| `productId` | Granted product |

Adopt a principle on this product, or stop adopting it. `principleId` is the principle human id or uuid. `action` is `adopt` or `stop`. `mode` defaults to `suggest`. `suggest` or `propose` files one Review proposal with `targetType` `principle_adoption`: `kind` `add` for adopt, `kind` `obsolete` for stop. `mode` `edit` writes the row when this product is in `editDirectlyProductIds` or the credential has edit scope, and is **403** without that grant. Only a principle recorded in this product can be adopted; anything else is **422**. A second active adoption is **409**. Stop sets `ended_at` and keeps the row. It does not change the principle, its revisions, or its links. Stopping a tenant adoption is **403**.

### Principle conflicts

`ih_list_principle_conflicts`. Read.

| Argument | Value |
| --- | --- |
| `unresolved` | When true, only pairs that apply here and have no recorded priority |
| `productId` | Granted product |

Conflicts recorded for this product and the tenant. Read. `unresolved` true returns only pairs where both principles apply to this product, a live conflict exists at this product or the tenant, and no live row at this product or the tenant has a priority. A tenant priority resolves the pair for every product. Each entry includes the two principles, the priority when one is recorded, the reason, and scope `product` or `tenant`.

`ih_propose_principle_conflict`. Suggest or edit.

| Argument | Value |
| --- | --- |
| `principleA` | Required. Principle human id or uuid |
| `principleB` | Required. Principle human id or uuid |
| `priorityPrincipleId` | Optional. One of the two principles. Records which one takes priority |
| `reason` | Required when `priorityPrincipleId` is set. Why that principle takes priority |
| `mode` | `suggest`, `propose`, or `edit`. Defaults to `suggest` |
| `productId` | Granted product |

Mark two principles as conflicting, or record which one takes priority. `principleA` and `principleB` are human ids or uuids. They are stored in uuid order, so A,B and B,A are the same pair. `priorityPrincipleId` is optional and must be one of the pair. `reason` is required when a priority is set. `mode` defaults to `suggest`. `suggest` or `propose` files one Review proposal with `targetType` `principle_conflict`. Accept applies it. `mode` `edit` writes the row when this product is in `editDirectlyProductIds` or the credential has edit scope, and is **403** without that grant. A second live mark of the same pair is **409**.

## Confirmation review

`ih_record_confirmation_review`. REST: `POST /api/requirements/{id}/confirmation-reviews`.

| Argument | Value |
| --- | --- |
| `id` | Product Requirement human id or uuid |
| `revision` | Revision number |
| `recordKind` | `implementation`, `deployment_event`, or `test` |
| `recordId` | Implementation or test human id or uuid, or the deployment completed event uuid |
| `evidenceUrl` | Optional http(s) link |
| `productId` | Granted product, when you are naming one |

The same revision, record, and evidence returns the existing row. Pass `productId` when the credential can open more than one product.

## Status assessment

`ih_record_status_assessment`. REST: `POST /api/requirements/{id}/status-assessments`.

| Argument | Value |
| --- | --- |
| `id` | Product Requirement human id or uuid |
| `revision` | Revision number |
| `implementationCompleteness` | `unimplemented`, `partially_implemented`, or `fully_implemented` |
| `testCoverage` | `untested`, `partially_tested`, or `comprehensively_tested` |
| `completenessComment` | Optional |
| `coverageComment` | Optional |
| `evidenceUrl` | Optional http(s) link |
| `productId` | Granted product, when you are naming one |

An identical latest assessment for that revision returns the existing row. Say verdicts to a person in words. Pass the snake_case values to the tool. If the tool schema on the call lists different spellings, use the schema.

When to confirm, what counts as complete, and what counts as tested are PRD-180 in the developer plugin. This catalog only names the calls.

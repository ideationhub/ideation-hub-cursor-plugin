# REST API and lifecycle reference

This is the canonical REST reference for Ideation Hub. The developer and requirements skills point here. Load this file when you call the REST API directly or need the exact behavior of a lifecycle action. The MCP catalog (`mcp.md`) covers the same operations. Every route below is also mapped under `/api/v0/...`.

## OAuth 2.0 `client_credentials` token

`POST /api/oauth/token` (also `/api/v0/...`; every route below is dual-mapped):

```json
{ "grant_type": "client_credentials", "client_id": "$IH_CLIENT_ID",
  "client_secret": "$IH_CLIENT_SECRET", "productId": "<uuid, optional>" }
```

- **200** `{ access_token, token_type: "Bearer", expires_in, productId }`. Omit `productId` for the credential's home Product; a sent one must be on its allow-list.
- **401** wrong secret / revoked. Re-mint when the token expires.
- Send `Authorization: Bearer <token>`, or call MCP with `X-IH-Client-Id` and `X-IH-Client-Secret` and skip the token. The token pins tenant + Product. MCP may pass `productId` only when that product is on the credential grant; otherwise **403**.
- Permissions: `read`, `suggest` (stored as `propose`), and direct edit via `editDirectlyProductIds` or an `edit` scope. Missing permission → **403**. `review` is not a scope.

## Read calls

| Need | Call |
| --- | --- |
| All requirements on the Product | `GET /api/requirements` |
| One requirement | `GET /api/requirements/{id}` (human id or UUID) |
| Neighbors up/down the trace | `GET /api/requirements/{id}/neighbors?direction=up\|down` |
| TraceLinks | `GET /api/trace-links` |
| Profiles | `GET /api/products/{productId}/profiles`, `GET /api/profiles/{id}` |
| Full Profile → UN → PRD chain | `GET /api/products/{productId}/chain` (or `GET /api/chain`) |

## Change calls

| Change | Call | Notes |
| --- | --- | --- |
| Add BN/UN/PRD | `POST /api/requirements` | `type`: `business_need`\|`user_need`\|`product_requirement`; `title`, `statement`. UN needs `profileId` or a `profileName` matching exactly one canonical Profile. PRD needs `unId` of a canonical UN; Accept writes the `addresses` link. |
| Edit | `POST /api/requirements/{id}/revisions` | `{ title?, statement?, proposalId?, addLinks?, removeLinks?, addProfileIds?, removeProfileIds? }`. The one pending add or edit on that requirement is updated in place, whoever filed it. `addLinks` and `removeLinks` are arrays of `{type, targetId \| targetProfileId}`. On a User Need, `addProfileIds` and `removeProfileIds` are shorthand. They are stored on the one pending edit, shown in `pendingProposal.payload`, and applied in the same transaction at Accept. A conflict at Accept is **409** `link_conflict` or `duplicate_link` and leaves the requirement unchanged. A new proposal is filed only when none is pending. Filing a second is **409** `pending_proposal_exists`. Legacy duplicates are **409** `ambiguous_proposal` (pass `proposalId`). |
| Link | `POST /api/trace-links` | `{ sourceId, type, targetId? \| targetProfileId? }`. Pass exactly one target. A User Need traces to a User Profile with type `addresses` and `targetProfileId` set to the profile human id (for example `UP-004`); do not also pass `targetId`. Suggest, exactly one pending-add end: stored with that requirement (`outcome: "bundled"`, `bundledWith`). No standalone Review item. Suggest, both ends existing: `kind=link` proposal; Accept adds only the named profile and keeps the others. Direct edit writes the row. Two pending adds stay a standalone proposal. **422** `link_pair_not_allowed`, `profile_not_in_product`, or `link_target` (both or neither target given). **409** `duplicate_link`. Proposing the same pending link again returns the existing proposal. |
| Unlink | `POST /api/trace-links/remove` | Soft-remove a User Need to User Profile link with `sourceId`, type `addresses`, and `targetProfileId`. Same applied-versus-proposed split as Link. To remove a requirement-to-requirement link, use `POST /api/trace-links/{id}/remove`. |
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

## Status, confirmation, and development-phase routes

These are direct writes or reads. They are not proposals. `status:write` never edits a title, a body, or a trace link. `delivery:write` records a development-phase event.

| Need | Call |
| --- | --- |
| Derived and assessed status for each revision | `GET /api/requirements/{id}/revision-status` |
| Confirm an earlier implementation, deployment event, or test still applies to this revision | `POST /api/requirements/{id}/confirmation-reviews` |
| Record an implementation-completeness and test-coverage assessment for one revision | `POST /api/requirements/{id}/status-assessments` |

The MCP arguments for those calls are in `mcp.md`. How to judge a Product Requirement (PRD-180) stays in the developer skill's `references/assessment.md`. This file does not add judgment rules.

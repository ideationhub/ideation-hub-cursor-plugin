# Write protocol

The rules here are grounded in the Ideation Hub MCP contract. Read the requirements skill for credentials and the general proposal rules; this file adds the design-specific protocol.

## What decides whether a write is live

**The credential decides, not the tool name.**

| Credential (from `ih_list_products`) | A write call without `mode` | With `mode: "suggest"` |
| --- | --- | --- |
| Read only | 403 | 403 |
| Suggest (`directEdit: false`) | pending proposal in the issuer's Review queue | pending proposal |
| Edit (`directEdit: true`) | **changes the living record immediately** | pending proposal |

This applies to `ih_propose_requirement`, `ih_propose_edit`, `ih_propose_bulk_push`, `ih_propose_profile`, `ih_link_requirements`, `ih_unlink_requirements`, `ih_lifecycle_action` and the section tools. `ih_create_requirement`, `ih_update_requirement` and `ih_set_tags` are direct-only.

## Rule: always suggest

Pass `mode: "suggest"` on every write. Use `mode: "edit"` only when the user, in this conversation, explicitly asks for a direct write to a named product, and `directEdit` is true for it. Repeat back what will change before doing it.

Never call `ih_create_requirement`, `ih_update_requirement` or `ih_set_tags`.

## Sequence for every write

1. **Read.** `ih_get_requirement` on each record you will change, and on each link target. Note `revision`, `status`, `tags` and `pendingProposal`.
2. **Plan.** List each change: new record, edit, link add or remove, lifecycle action. Give a one-line reason and the justifying IDs for each.
3. **Confirm.** Show the plan. Changes the user explicitly asked for need no second confirmation. Changes you recommend, and every change to an accepted record, need the user's yes.
4. **Dry run.** For new records, call `ih_propose_bulk_push` with `dryRun: true`, `mode: "suggest"` and the same items and links you will file. A dry run stores nothing and uses no human IDs. It checks types, required fields, duplicate titles and that parents and link ends exist. It does **not** check link direction; do that yourself (`traceability.md`). Single-record edits can use `ih_validate_against_trace` with `proposedPayload`.
5. **File.** Repeat the call without `dryRun`, with `mode: "suggest"`. For edits, use `ih_propose_edit` with `mode: "suggest"` and `expectedRevision`.
6. **Report.** For each result: proposal `id`, kind (add, edit, link, obsolete…), reserved human ID or target ID, link outcome (`bundled`, `proposed`, `applied`), and the `reviewUrl`. Say that a person must Accept on www.ideationhub.me (avatar → Agent proposals, or the product's Review tab).

## Bulk push shape

```json
{
  "mode": "suggest",
  "dryRun": true,
  "items": [
    {"ref": "ctx", "type": "SD", "title": "System context: ShopTrack", "statement": "…", "tags": ["arch.context"], "parentId": "PRD-003"},
    {"ref": "api", "type": "CD", "title": "Shopping list API", "statement": "…", "tags": ["arch.interface"], "parentId": "ctx"}
  ],
  "links": [
    {"sourceId": "PRD-004", "type": "realized_by", "targetId": "ctx"}
  ]
}
```

- `parentId` on an SD, UXD or CD writes `parent realized_by item`. Use it for the first justifying link, and `links` for the rest. Never list the same link twice.
- `ref` values let later items and links name earlier items in the same call.
- A real (non-dry) write requires `statement` on every item.
- Titles must be unique within a type in the product.

## Changing records

| Situation | Do |
| --- | --- |
| The record has a `pendingProposal` (`canAmend: true`) | Amend it: `ih_propose_edit` with `proposalId`. Never file a second proposal. Anyone with propose permission may amend, so read its `payload` first and keep others' changes unless the user agrees to drop them |
| The record is accepted, no pending proposal | `ih_propose_edit` with `mode: "suggest"` and `expectedRevision`, after the user confirms |
| 409 `pending_proposal_exists` or `ambiguous_proposal` | Read the named proposal(s) and amend with `proposalId` |
| 409 on `expectedRevision` | Someone changed it. Re-read, re-plan, re-confirm |
| 409 `duplicate_link` | The link exists already. Treat as done; do not retry |
| 409 `link_conflict` at accept | Tell the user; the reviewer must resolve it |
| 422 `link_pair_not_allowed`, `link_target`, `profile_not_in_product` | Your link is wrong. Fix the plan; never retry unchanged |
| 422 no-op | Nothing changed; do not file |
| 422 `duplicate_title` | Choose a distinct title, or extend the existing record instead |
| 403 | You lack permission. Say so plainly and stop that write. Never retry with another tool or credential |

`tags` on an edit replaces the whole list. Send the full list, or omit `tags` to leave it unchanged.

## Retiring records

- Never hard-delete; Ideation Hub never hard-deletes and human IDs are never reused.
- To retire an accepted design, propose `ih_lifecycle_action` `action: "obsolete"`, `mode: "suggest"`, with a `reason`, after the user confirms.
- There is no supersede action. To replace a decision, file the new decision SD, move links with `addLinks`/`removeLinks`, and propose `obsolete` on the old one.
- To withdraw your own pending proposal, use `withdraw_proposal` with the **proposal id**. Use `withdraw_requirement` only when the user asks for exactly that.
- Never re-propose to get around a Reject. Ask the user.

## Direct write: `design_ready`

At handoff, call `ih_record_development_phase_event` with `phase: "design_ready"` for each in-scope Product Requirement, with `evidenceUrl` set to the design's Review URL or record URL if you have one. This is a direct, append-only write by design; it is not requirement content, and retries with the same arguments return the same event. It requires the `delivery:write` permission. On 403, tell the user: "This credential cannot record development-phase events (it lacks delivery:write), so 'design ready' was not recorded." Then continue. Do not try other tools or credentials.

Confirmation reviews and status assessments are also direct writes. They are not design writes and they are not proposals. Follow `assessment.md`. They require `status:write`, and that permission never changes a title, body, or trace link. On 403, tell the user the credential lacks `status:write` and continue. Do not try other tools or credentials.

## Safety matrix

| Action | Suggest credential | Edit credential (this skill's behaviour) |
| --- | --- | --- |
| Read, search, validate, dry run | allowed | allowed |
| New SD/CD/UXD and its links | proposal | proposal (`mode: "suggest"`) |
| Change a pending design | amend its proposal | amend its proposal (a direct edit would 409) |
| Change an accepted design | edit proposal, after confirmation | edit proposal, after confirmation; direct only on explicit request |
| Change a requirement (BN/UN/PRD) or a test's wording | not done by this skill; hand to the requirements skill | same |
| Obsolete, withdraw, reclassify | proposal, after confirmation | proposal, after confirmation |
| `design_ready` event | direct; 403 without `delivery:write` | direct; 403 without `delivery:write` |
| Confirmation review or status assessment | direct; 403 without `status:write`; never changes title, body, or links | direct; 403 without `status:write`; never changes title, body, or links |
| Accept or Reject | never (403) | never (403) |

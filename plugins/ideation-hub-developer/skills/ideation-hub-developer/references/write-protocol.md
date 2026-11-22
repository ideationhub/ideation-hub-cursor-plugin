# Write protocol

How a credential turns a write into a proposal or a living change, the tool arguments, the proposal outcomes (`bundled`, `proposed`, `applied`), and the error codes are defined once in the platform plugin. Do not restate them.

- `../../../../ideation-hub-platform/references/mcp.md` — tools and the write rule
- `../../../../ideation-hub-platform/references/rest-api.md` — the same operations over REST
- `../../../../ideation-hub-requirements/skills/ideation-hub-requirements-agent/SKILL.md` — authoring workflow, when a requirement's wording must change

This file is the extra rule for design and implementation writes.

## Rule: always suggest

Pass `mode: "suggest"` on every design or implementation write (`ih_propose_requirement`, `ih_propose_edit`, `ih_propose_bulk_push`, `ih_link_requirements`, `ih_unlink_requirements`, `ih_lifecycle_action`). Use `mode: "edit"` only when the user, in this conversation, explicitly asks for a direct write to a named product, and `directEdit` is true for it. Repeat back what will change before doing it.

Never call `ih_create_requirement`, `ih_update_requirement` or `ih_set_tags`.

Never author or edit a Business Need, User Need, Product Requirement, or a test's acceptance wording. Hand that to the requirements plugin.

## Sequence for every design write

1. **Read.** `ih_get_requirement` on each record you will change, and on each link target. Note `revision`, `status`, `tags` and `pendingProposal`.
2. **Plan.** List each change: new record, edit, link add or remove, lifecycle action. Give a one-line reason and the justifying IDs for each.
3. **Confirm.** Show the plan. Changes the user explicitly asked for need no second confirmation. Changes you recommend, and every change to an accepted record, need the user's yes.
4. **Dry run.** For new records, call `ih_propose_bulk_push` with `dryRun: true`, `mode: "suggest"` and the same items and links you will file. A dry run stores nothing. It does **not** check link direction; do that yourself (`traceability.md`). Single-record edits can use `ih_validate_against_trace` with `proposedPayload`.
5. **File.** Repeat the call without `dryRun`, with `mode: "suggest"`. For edits, use `ih_propose_edit` with `mode: "suggest"` and `expectedRevision`.
6. **Report.** For each result: proposal `id`, kind (add, edit, link, obsolete…), reserved human ID or target ID, link outcome, and the `reviewUrl`. Say that a person must Accept on www.ideationhub.me (avatar → Agent proposals, or the product's Review tab).

On 409 or 422, follow the platform `mcp.md`: amend the pending proposal, re-read after an `expectedRevision` mismatch, and do not retry an unchanged call. On 403, say which permission is missing and stop that write. Never work around it with another tool.

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

- `parentId` on an SD, UXD or CD writes `parent realized_by item`, as the platform schema states. Use `parentId` for the first justifying link, and `links` for the rest. Never list the same link twice.
- `ref` values let later items and links name earlier items in the same call.
- A real (non-dry) write requires `statement` on every item.
- Titles must be unique within a type in the product.

`tags` on an edit replaces the whole list. The tag format is in the platform schema. Send the full list, or omit `tags` to leave it unchanged.

## Retiring records

Follow the platform `mcp.md` and `rest-api.md` for `obsolete`, `withdraw_proposal`, and `withdraw_requirement`. Requirements are never hard-deleted, and human IDs are never reused.

There is no supersede action. To replace a decision, file the new decision SD, move links with `addLinks`/`removeLinks`, and propose `obsolete` on the old one after the user confirms.

To withdraw your own pending proposal, use `withdraw_proposal` with the **proposal id**. Use `withdraw_requirement` only when the user asks for exactly that. Never re-propose to get around a Reject. Ask the user.

## Direct writes that are not proposals

At handoff, call `ih_record_development_phase_event` with `phase: "design_ready"` for each in-scope Product Requirement, with `evidenceUrl` set to the design's Review URL or record URL if you have one. The tool is in the platform `mcp.md`. This use of it is a direct, append-only write; it is not requirement content, and retries with the same arguments return the same event. It requires `delivery:write`. On 403, tell the user: "This credential cannot record development-phase events (it lacks delivery:write), so 'design ready' was not recorded." Then continue. Do not try other tools or credentials.

Confirmation reviews and status assessments are also direct writes. They are not design writes and they are not proposals. The calls are in the platform `mcp.md`. When to use them, and how to judge, is `assessment.md` only. They require `status:write`, and that permission never changes a title, body, or trace link. On 403, tell the user the credential lacks `status:write` and continue. Do not try other tools or credentials.

## What this skill will and will not do

Permission and outcome meanings are in the platform `mcp.md`. This is the behavior of this plugin:

| Action | What this skill does |
| --- | --- |
| Read, search, validate, dry run | allowed |
| New SD/CD/UXD, IMPL, or test link | proposal (`mode: "suggest"`) |
| Change a pending design | amend its proposal |
| Change an accepted design | edit proposal, after confirmation; direct only on an explicit request |
| Change a requirement (BN/UN/PRD) or a test's wording | not done here; hand it to the requirements plugin |
| Obsolete, withdraw, reclassify a design | proposal, after confirmation |
| `design_ready` event | direct; 403 without `delivery:write` |
| Confirmation review or status assessment | direct, as `assessment.md` describes; 403 without `status:write`; never changes title, body, or links |
| Accept or Reject | never |

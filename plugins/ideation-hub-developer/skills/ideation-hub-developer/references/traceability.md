# Traceability for designs

Ideation Hub's trace links are the only traceability store. Never keep a separate trace table, spreadsheet or file as the authority.

Link type names, stored direction, allowed pairs, revision pins, and how to file a link are defined once in the platform plugin. From this file:

- `../../../../ideation-hub-platform/references/schema.md`
- `../../../../ideation-hub-platform/references/mcp.md`
- `../../../../ideation-hub-platform/references/rest-api.md`

This plugin does not create `addresses` links. A missing or wrong User Need link is proposed through the requirements plugin's authoring workflow (`implementation.md`), not written here.

Check direction yourself before every link, using the platform schema. Never create a link the table below does not allow.

## Design links this skill creates

| Meaning | Link (source → target) |
| --- | --- |
| A design delivers a requirement (including an NFR) | `PRD realized_by SD`, `PRD realized_by UXD` |
| A container or decision details the context | `SD refined_by SD` |
| A decision shapes an element | `SD(decision) refined_by SD` or `SD(decision) refined_by CD` |
| A component belongs to a system or container | `SD realized_by CD` |
| A sub-component, interface or data model details a component | `CD refined_by CD` |
| An implementation delivers a component or system | `CD realized_by IMPL`, or `SD realized_by IMPL` when there is no CD. Record it as `SKILL.md` (Implementation) describes when this skill is asked to implement |

<!-- Test-link rule: the single place this skill states it. Change only this line if the rule changes. -->
**Test links:** a design is verified by `SD checked_by VER`, `CD checked_by VER` or `CD checked_by UT`. Verification Tests (VER) check Product Requirements, designs and implementations; Validation Tests (VT) check User Needs and Business Needs, so a design is never `checked_by` a VT.

## Filing a design link

File the link the way the platform schema and `mcp.md` describe (`parentId`, `links` on a bulk push, `addLinks` / `removeLinks`, or `ih_link_requirements` / `ih_unlink_requirements`), with `mode: "suggest"`. `parentId` on an SD, UXD, or CD writes `parent realized_by` the new record. Do not also put that same link in `links`. A bulk dry run does not check direction. Check it yourself.

## Revision pins

Revision pins and staleness are defined in the platform schema. Use them to find designs affected by a requirement change.

## Checks you run (Phase E and after any change)

Run these from Ideation Hub data (`ih_get_chain` with `includePending: false` for accepted state, and again with `includePending: true` for the state including your proposals). Report each result as **verified**, because it comes straight from data.

1. **Coverage:** every in-scope PRD (including NFRs) has at least one outbound `realized_by` to an SD, UXD or CD. List the PRDs with none.
2. **Justification:** every SD, UXD and CD has at least one inbound `realized_by` or `refined_by` from a PRD or a broader design. List orphans. An orphan is either justified (add the missing link) or unnecessary (propose `obsolete` after the user confirms).
3. **Direction:** every link touching a design matches the table above. List violations.
4. **Staleness:** every link touching a design pins the current revision of both ends. List stale links with old and new revision.
5. **Open items:** every design record still tagged `arch.assumption` or `arch.open-question`.
6. **NFR response:** every in-scope PRD tagged `NFR` or `principle` is `realized_by` at least one design record whose body states the response.
7. **Verification:** in-scope PRDs and CDs with no `checked_by` link. Report as a gap; tests are authored through the requirements skill.

## Impact analysis

When a requirement changes (new revision, obsolete or withdrawn):

1. `ih_get_neighbors` from it, `direction: "down"`, `depth: 3`.
2. For each design reached, read the link pin. A pin older than the current revision means the design was reviewed against old text.
3. Compare the old and new requirement revisions (`revisions[]` on `ih_get_requirement`) and decide per design: no change needed, body change, or structural change.
4. Tell the user the list, with your proposed change for each. File changes as proposals after they agree.
5. If a design stays valid unchanged, say so; the link pin stays as is until someone re-links it at the new revision (pass `targetRevision` on `ih_link_requirements` only if the user asks to re-pin).

When a design decision changes, walk its outbound `refined_by` links the same way.

## Never

- Never invent a link to make coverage look complete.
- Never link a design to a pending requirement without labelling it a proposal.
- Never remove a link without saying why and getting agreement; removal is a proposal too.

# Example 4 — Iterating: a changed decision, a changed requirement, review and handoff

*Illustrative transcript. IDs and results are invented for the example.*

**Scenario:** SD-001 (context), SD-002 (decision "Use PostgreSQL full-text search for product search"), CD-001 (Search API) and CD-002 (Catalogue data model) are accepted. The user now wants a search service, and PRD-030 has a new revision.

---

**User:** We're adding typo tolerance — PRD-030 now requires it. Revisit the search decision.

**Agent** (calls `ih_get_requirement PRD-030` → revision 2, `pendingProposal: null`; `ih_get_neighbors PRD-030 direction: down` → links pinned at PRD-030@1 to SD-002 and CD-001; reads SD-002's outbound `refined_by` → CD-001, CD-002):

> **Impact (verified from links):** PRD-030 moved from @1 to @2. Two designs were reviewed against @1: the search decision (SD-002) and the Search API (CD-001). The Catalogue data model (CD-002) is unaffected apart from needing a change feed, which I'll add.
>
> **What changes:**
> 1. New decision SD "Use a managed search service for product search", superseding SD-002. Options re-run with typo tolerance as a driver; managed search now wins on PRD-030@2.
> 2. Propose `obsolete` on SD-002, and move its `refined_by` links to the new decision.
> 3. Edit CD-001 (Search API): the provider becomes the search service; the interface stays the same, so no consumer changes.
> 4. Edit CD-002: add a change feed for index updates.
>
> **Unchanged:** SD-001 context (still one deployable app plus managed services), every other component.
> **Downstream:** the PRD-091 latency load test still applies; any IMPL linked to CD-001 will need a code change.
>
> These change accepted records, so I need your yes before filing. OK?

**User:** Yes.

**Agent** (calls `ih_propose_bulk_push` with `dryRun: true`, `mode: "suggest"` for the new decision SD, whose `links` include `refined_by` to CD-001 and CD-002 → valid; files it the same way without `dryRun`; `ih_propose_edit CD-001 {mode: "suggest", expectedRevision: 1, statement: …}`; `ih_propose_edit CD-002 {mode: "suggest", expectedRevision: 1, …}`; `ih_lifecycle_action {id: "SD-002", action: "obsolete", reason: "Superseded by the managed search decision", mode: "suggest"}`. SD-001 is unchanged, so it is not touched):

> Filed for review:
> - add SD-005 "Use a managed search service for product search" (links bundled)
> - edit CD-001 Search API (proposal 91a2…)
> - edit CD-002 Catalogue data model (proposal 91a4…)
> - obsolete SD-002 (proposal 91a6…)
>
> SD-002 stays accepted until you Accept the obsolete proposal.

**User:** Accepted all. Review the design and hand it off.

**Agent** (reads the chain for the `search` tag, `includePending: false`):

> **Validation — verified:** coverage 3/3 PRDs; no orphans; all link directions legal; no stale pins; no open questions. **Gap (verified):** PRD-030@2 has no `checked_by` test for typo tolerance.
> **Findings:** minor, judgement — the index refresh delay should be stated in CD-001's rules.
>
> **Handoff:** brief built from accepted records (SD-001, SD-005, CD-001, CD-002; PRD-030@2, PRD-091@1). Recorded `design_ready` on PRD-030 and PRD-091 with `ih_record_development_phase_event`.
>
> *(If the credential lacked `delivery:write`, the agent would instead say: "This credential cannot record development-phase events, so 'design ready' was not recorded," and still deliver the brief.)*

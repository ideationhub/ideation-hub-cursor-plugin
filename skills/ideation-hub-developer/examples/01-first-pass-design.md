# Example 1 — First-pass design for a small app

*Illustrative transcript. The product, IDs and tool results are invented for the example; the record formats and tool calls match the Ideation Hub MCP contract.*

**Scenario:** a small price-tally phone app. Six accepted PRDs, two NFRs, one existing tag `tally`.

---

**User:** Design the system for everything tagged `tally` in Shopping Price Tally.

**Agent** (calls `ih_list_products` → `directEdit: false`; `ih_search_requirements {query: "", tags: ["tally"]}`; `ih_get_neighbors` up on each PRD; `ih_search_requirements {tag: "NFR"}`; `ih_search_requirements {types: ["SD","CD","UXD"]}` → none):

> **Problem frame** (read just now; all accepted)
> - Purpose: shoppers keep a running total of shelf prices while shopping (UN-001@1, BN-001@1).
> - Functional scope: scan a price tag and confirm the price (PRD-010@2), keep one active tally with item count (PRD-011@1), edit or remove an item (PRD-012@1), keep a trip across app restarts and stores (PRD-013@1), show an optional tax estimate (PRD-014@1), sell 10-trip credit packs (PRD-015@3).
> - Quality drivers: scanning and tallying work offline (PRD-020@1, NFR); price recognition runs on the phone (PRD-021@1, NFR).
> - Constraints stated: on-device recognition and storage (PRD-021).
> - Missing: target platforms (iOS, Android or both); whether credits must survive a reinstall. I'll assume **iOS and Android, credits recoverable after reinstall** (provisional).
>
> **First pass — rung 1 (small).** One mobile app plus the app stores' purchase services, and a very small backend only for credit recovery.
>
> 1. **System context: Shopping Price Tally** (SD, `arch.context`). Inside: the mobile app and a credit ledger service. Outside: shopper, App Store / Google Play billing. Style: offline-first mobile client because PRD-020 requires offline tallying; a backend exists only because credits must be recoverable (provisional assumption), which on-device storage alone cannot give.
> 2. **Tally engine** (CD, `arch.component`): items, quantities, subtotal, tax estimate; local storage. Delivers PRD-011, -012, -013, -014.
> 3. **Price capture** (CD, `arch.component`): on-device text recognition with a confirm/correct step. Delivers PRD-010; responds to PRD-021. *Buy:* the platform's on-device text recognition rather than a custom model.
> 4. **Credit ledger** (CD, `arch.component` + `arch.data`): validates store receipts and stores the credit balance per account. Delivers PRD-015. *Buy:* store billing and a managed receipt-validation service are options; recommended: platform billing plus a small serverless function and a managed database.
>
> Not added, on purpose: user accounts beyond what the store purchase identity gives, a sync service for tallies (PRD-013 keeps trips local), analytics.
>
> **Two questions that change the design:**
> 1. *Must purchased credits survive a reinstall or a new phone?* If yes, we need the credit ledger backend. If no, credits can live on the device and the design has no backend at all. **Recommendation:** yes, because losing paid credits would generate support work. I'm assuming yes for now.
> 2. *iOS, Android or both at launch?* It changes the price-capture component (two recognition SDKs) and billing integration. **Recommendation:** both via one cross-platform codebase, unless you plan a single-platform pilot.
>
> If you're happy with the direction, I'll do a dry run of these four records and then file them as proposals for your review.

**User:** Credits must survive reinstall. Both platforms. Go ahead.

**Agent** (calls `ih_propose_bulk_push` with `dryRun: true`, `mode: "suggest"`, four items with refs `ctx`, `tally`, `capture`, `ledger`; `parentId` PRD-011 on `ctx`; links `PRD-010/012/013/014/015/020/021 realized_by ctx`, `ctx realized_by tally|capture|ledger`, `PRD-011 realized_by tally`, … → `{"dryRun": true, "items": [], "links": []}` with no errors. Then the same call without `dryRun`.):

> Filed for review (nothing changes until you Accept):
>
> | Proposal | Kind | Record | Links |
> | --- | --- | --- | --- |
> | 7f1e… | add | SD-001 System context: Shopping Price Tally | 7 × `realized_by` from PRDs — bundled |
> | 7f20… | add | CD-001 Tally engine | bundled |
> | 7f22… | add | CD-002 Price capture | bundled |
> | 7f24… | add | CD-003 Credit ledger | bundled |
>
> Review them at www.ideationhub.me → Agent proposals. The two assumptions you just confirmed are recorded as *confirmed* in SD-001's Assumptions section.

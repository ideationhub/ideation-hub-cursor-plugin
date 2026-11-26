# Example 1 — First-pass design for a small app

*Illustrative transcript. Requirement IDs and tool results are invented for the example; the record formats and tool calls match the Ideation Hub MCP contract. The stack, pricing, size, and store facts follow the ShopTally notes on main: [architecture decisions](https://github.com/alexworden/IdeationHub/blob/main/docs/products/shoptrack/architecture/architecture-decisions.md) (ADR-002, 2026-11-24) and the [23 November 2026 research summary](https://github.com/alexworden/IdeationHub/blob/main/docs/products/shoptrack/architecture/research-summary-2026-11-23.md).*

**Scenario:** a small price-tally phone app. Six accepted PRDs, two NFRs, one existing tag `tally`. The mobile stack is already decided.

---

**User:** Design the system for everything tagged `tally` in Shopping Price Tally.

**Agent** (calls `ih_list_products` → `directEdit: false`; `ih_search_requirements {query: "", tags: ["tally"]}`; `ih_get_neighbors` up on each PRD; `ih_search_requirements {tag: "NFR"}`; `ih_search_requirements {types: ["SD","CD","UXD"]}` → none):

> **Problem frame** (read just now; all accepted)
> - Purpose: shoppers keep a running total of shelf prices while shopping (UN-001@1, BN-001@1).
> - Functional scope: scan a price tag and take the lowest price on that tag by default — the shopper can change it, with no forced confirmation (PRD-010@2); keep one active tally with item count (PRD-011@1); edit or remove an item (PRD-012@1); keep a trip across app restarts and stores (PRD-013@1); show an optional tax estimate (PRD-014@1). The $1.99 paid download includes the first 10 tallies. A tally credit is used up on the first scan of a trip. Later packs of 10 are consumable in-app purchases (PRD-015@3).
> - Quality drivers: scanning and tallying work offline (PRD-020@1, NFR); price recognition runs on the phone (PRD-021@1, NFR). Download budgets: at most 30 MB on iPhone and 25 MB on Android, well under Play's 200 MB warning threshold and Apple's 200 MB cellular prompt. Startup budget: stay inside Android vitals, which flags a cold start of 5 seconds or more, a warm start of 2 seconds or more, and a hot start of 1 second or more ([Android vitals](https://support.google.com/googleplay/android-developer/answer/9844486?hl=en)). Apple publishes no startup number in the pages read.
> - Constraints stated: on-device recognition and storage (PRD-021). **ADR-002 (2026-11-24), accepted:** native Swift for iOS and Kotlin for Android, chosen for the lowest binary-size and startup risk. Development and user testing are iPhone first. Design covers both platforms side by side. This supersedes the earlier Flutter direction (ADR-001).
> - Store ceilings, for scale only: Apple's uncompressed app maximum is about 4 GB, and App Store Connect warns when a device variant's download exceeds the 200 MB over-the-air cellular limit ([maximum build file sizes](https://developer.apple.com/help/app-store-connect/reference/app-uploads/maximum-build-file-sizes/), [view builds and metadata](https://developer.apple.com/help/app-store-connect/manage-builds/view-builds-and-metadata/)). On Google Play, above 200 MB people on mobile data see a non-blocking size warning; that 200 MB figure is also the historical base-module limit. The current base-module limit is 500 MB, and modules plus install-time asset packs share a 4 GB cap ([Play size limits](https://support.google.com/googleplay/android-developer/answer/9859372?hl=en)).
> - Missing: whether credits must survive a reinstall. I'll assume **credits recoverable after reinstall** (provisional). Platforms are not missing: ADR-002 already covers both.
>
> **First pass — rung 1 (small).** Two native apps plus the stores' purchase services, and a very small backend only for credit recovery. iPhone is built and tested first; the same design is drawn for Android beside it.
>
> 1. **System context: Shopping Price Tally** (SD, `arch.context`). Inside: the iOS app, the Android app, and a credit ledger service. Outside: shopper, App Store / Google Play billing. Style: offline-first native clients because PRD-020 requires offline tallying. A backend exists only because credits must be recoverable (provisional assumption), which on-device storage alone cannot give. Two codebases are the accepted cost of keeping download size and startup predictable.
> 2. **Tally engine** (CD, `arch.component`): items, quantities, subtotal, tax estimate; local storage. One credit is used on the first scan of a trip. Delivers PRD-011, -012, -013, -014.
> 3. **Price capture** (CD, `arch.component`): on-device text recognition. The line defaults to the lowest price on the scanned tag; the shopper can change it; there is no forced confirmation. Delivers PRD-010; responds to PRD-021. *Buy:* Apple Vision on iOS (built into the OS, 0 MB) and bundled ML Kit Latin on Android (about 4 MB, so scanning works offline). Not a custom model, and not ML Kit on iOS.
> 4. **Credit ledger** (CD, `arch.component` + `arch.data`): proves the paid download for the included 10 tallies, validates consumable repeat-pack receipts, and stores the remaining balance. Credits do not expire. Delivers PRD-015. *Buy:* `AppTransaction` on iOS 16+ and Play's licence verdict on Android for the download; store billing for repeat packs; a small serverless function and a managed database for the balance.
>
> Not added, on purpose: user accounts beyond what the store purchase identity gives, a sync service for tallies (PRD-013 keeps trips local), analytics, and a cross-platform engine.
>
> **Store rules to check**
> - Text recognition: Apple Vision on iOS adds 0 MB; bundled ML Kit Latin on Android adds about 4 MB and works offline. [Vision](https://developer.apple.com/documentation/vision/recognizing-text-in-images) · [ML Kit text recognition v2](https://developers.google.com/ml-kit/vision/text-recognition/v2/android)
> - Prove the download was paid: `AppTransaction` on iOS 16+; Play licensing reports only whether the account is licensed (`LICENSED`, `UNLICENSED`, or `UNEVALUATED`). [AppTransaction](https://developer.apple.com/documentation/storekit/apptransaction) · [Play Integrity verdicts](https://developer.android.com/google/play/integrity/verdicts)
> - Consumable credits bought in the app must not expire. [App Review Guideline 3.1.1](https://developer.apple.com/app-store/review/guidelines/#in-app-purchase)
> - Google Play refunds a purchase that is not acknowledged within 3 days. [Integrate Play Billing](https://developer.android.com/google/play/billing/integrate)
> - Reconcile store refunds and revocations; a store can still refund a paid download or a pack. [AppTransaction](https://developer.apple.com/documentation/storekit/apptransaction) · [Play billing security](https://developer.android.com/google/play/billing/security)
> - `NSCameraUsageDescription` must explain the camera use in the permission prompt (reading a price tag). [NSCameraUsageDescription](https://developer.apple.com/documentation/bundleresources/information-property-list/nscamerausagedescription)
> - App Store privacy nutrition labels: data processed only on the phone is not collected. [App privacy details](https://developer.apple.com/app-store/app-privacy-details/)
> - Google Play Data safety form is required even when the app collects nothing, plus a privacy-policy link; on-device processing is not declared. [Data safety](https://support.google.com/googleplay/android-developer/answer/10787469?hl=en)
> - From 27 January 2027, an app targeting Android 17 must use the location button for a one-off location request. [Permissions policy preview](https://support.google.com/googleplay/android-developer/answer/16909972?hl=en) · [Location permissions](https://developer.android.com/develop/sensors-and-location/location/permissions)
> - Size levers, as pointers only: [app thinning](https://developer.apple.com/help/app-store-connect/manage-builds/view-builds-and-metadata/), [Android App Bundles](https://developer.android.com/guide/app-bundle), [Play Asset Delivery](https://developer.android.com/guide/playcore/asset-delivery), and [On-Demand Resources](https://developer.apple.com/help/app-store-connect/reference/app-uploads/on-demand-resources-size-limits/) (deprecated as of iOS 27; Apple points to [Background Assets](https://developer.apple.com/documentation/backgroundassets)).
>
> **One question that changes the design:**
> 1. *Must purchased credits survive a reinstall or a new phone?* If yes, we need the credit ledger backend. If no, credits can live on the device and the design has no backend at all. **Recommendation:** yes, because losing paid credits would generate support work. I'm assuming yes for now.
>
> The platform choice is closed. ADR-002 selects native Swift and Kotlin, with iPhone-first development and user testing and both platforms designed side by side, in place of one cross-platform codebase.
>
> If you're happy with the direction, I'll do a dry run of these four records and then file them as proposals for your review.

**User:** Credits must survive reinstall. Go ahead.

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
> Review them at www.ideationhub.me → Agent proposals. The credit-recovery assumption you just confirmed, and ADR-002, are recorded as *confirmed* in SD-001's Assumptions section.

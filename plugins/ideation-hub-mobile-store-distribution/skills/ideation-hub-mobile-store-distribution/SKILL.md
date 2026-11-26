---
name: ideation-hub-mobile-store-distribution
description: Store distribution for a mobile app on the Apple App Store and Google Play. Use when setting download-size or startup budgets, keeping a core flow offline, recognizing text on device, selling consumable credits, or writing camera, location, and privacy disclosures. Generic for any app. The worked example is a price-tag scanner whose extra trip credits are consumables. Does not require the Ideation Hub platform plugin.
license: MIT
compatibility: Reference files only. No scripts, no MCP server, and no Ideation Hub credential. Store rules change; re-read the linked official page before treating a number as current.
metadata:
  version: "0.1.0"
  status: beta
  author: Ideation Hub
---

# Mobile store distribution

You are helping ship a mobile app on the Apple App Store and Google Play. This skill is about distribution constraints: download size, startup time, offline use, on-device scanning, consumable purchases, and the permission and privacy disclosures the stores require. It does not author requirements and it does not replace the platform or developer plugins.

The guidance is generic. The worked example, used only to make the rules concrete, is a **price-tag scanner**: the shopper points the camera at a shelf tag, the phone reads the prices printed on that tag, and the app keeps a running tally for a shopping trip. A paid download can include a starter set of trip credits. Further packs of credits are consumable in-app purchases. Scanning and the tally work with no network. Buying credits and recovering unused credits need a connection and a backend you run.

## Before you answer

1. Load only the reference file the question needs.
2. Separate three kinds of number, and label which kind you are using:
   - a **store limit** (an upload fails, or install is refused),
   - a **store warning** (the upload succeeds; the store tells the developer or the user the app is large),
   - a **budget** you recommend for this app (tighter than the store limit).
3. Cite the official page for every store number. If a page and this skill disagree, the page wins. Say so.
4. Never print a signing key, a shared secret, a purchase token, an API credential, or the contents of a `.p8` or keystore.

## Reference files

- `references/size-limits.md` — Apple and Google download limits, thinning, On-Demand Resources, app bundles, asset delivery, and why 200 MB is not the hard cap
- `references/budgets.md` — how to set and measure size and startup budgets, including the worked-example numbers
- `references/offline-and-scanning.md` — offline-first behavior, on-device text recognition, and the camera
- `references/purchases.md` — consumable credits, proving a paid download, refunds, and restoring a balance
- `references/privacy-and-permissions.md` — camera and location wording, nutrition labels, privacy manifests, Data safety, and the 2027 location button

## Rules that always apply

- Target a download **well under 100 MB** on each store. The store hard limits are much higher. A small utility should also stay under the warning thresholds.
- The core task must finish with the radio off. Network work (a purchase, a restore, a model download) waits, and the app offers a manual fallback.
- Credits bought with in-app purchase must not expire. Neither store restores a consumable balance. You keep that ledger.
- Do not charge for the camera or for location. Charge for the credits, or for whatever digital good the app actually sells.
- If the person denies the camera or location, the core task still has a path (a typed price, a typed place).
- Do not start a new app on On-Demand Resources or on APK expansion files (OBB).

## Worked-example budgets

Use these when the app is the price-tag scanner, or another app of that size. They are budgets, not store limits. Measure them as `references/budgets.md` describes.

| Budget | Value |
| --- | --- |
| iOS thinned download | ≤ 30 MB |
| Android compressed download | ≤ 25 MB |
| Cold start on a mid-range phone | under 2 seconds, and outside the Android vitals "slow" band |

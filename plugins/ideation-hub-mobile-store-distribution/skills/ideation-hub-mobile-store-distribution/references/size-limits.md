# Download size limits

Read this when the question is how large the binary may be, what happens above 200 MB, or whether expansion files or On-Demand Resources are still the way to ship extra bytes.

Label every number as a store limit, a store warning, or a budget. Budgets are in `budgets.md`. The recommendation for a normal app, including the price-tag scanner, is a download **well under 100 MB**. That is a budget. It is not a store cap.

Pages below were re-read on 26 November 2026. If a page has changed, the page wins.

## Apple

Two different limits are easy to mix up. The 4 GB figure is the maximum **uncompressed app size**. The 200 MB figure is the **over-the-air download** threshold App Store Connect warns on. 4 GB is not the cellular download cap.

| What | Number | Kind | Source |
| --- | --- | --- | --- |
| Maximum uncompressed app size, iOS 9 and later | 4 GB | Store limit | [Maximum build file sizes](https://developer.apple.com/help/app-store-connect/reference/app-uploads/maximum-build-file-sizes/) |
| Maximum executable size (`__TEXT` sections, total), iOS 9 and later | 500 MB | Store limit | same page |
| Any device variant whose download size exceeds this | 200 MB | Store warning, called the over-the-air download limit | [View builds and metadata](https://developer.apple.com/help/app-store-connect/manage-builds/view-builds-and-metadata/) |

[Reducing your app’s size](https://developer.apple.com/documentation/xcode/reducing-your-app-s-size) says the App Store limits how large an app can be when installed over a mobile connection, and that a person needs Wi-Fi once the app exceeds that limit. That page does not print the 200 MB number. The page that prints it is the build-metadata page: a yellow warning appears next to the build and next to the variant, and Apple emails you, when any variant’s download size exceeds the 200 MB over-the-air download limit.

This skill did not find a current `developer.apple.com` page that quotes the on-device prompt text. Do not invent the wording of that prompt. Do keep every thinned variant’s compressed download under 200 MB so a cellular install is not the case the warning describes. Well under 100 MB is the budget.

The final size is not the IPA you uploaded. After approval the binary is encrypted and recompressed, and it can grow. App Store Connect’s per-variant sizes, after processing, are the sizes to trust. The same build-metadata page says the exact final size cannot be known in advance.

### Thinning

App thinning is Apple building a smaller variant per device and OS. On the product page, iOS shows the install size for the device that is viewing the page. The universal variant is the one for older install paths (iOS 8 and earlier, volume purchase, older iTunes). Sources: [Maximum build file sizes](https://developer.apple.com/help/app-store-connect/reference/app-uploads/maximum-build-file-sizes/) and [View builds and metadata](https://developer.apple.com/help/app-store-connect/manage-builds/view-builds-and-metadata/).

Slicing is that per-device variant. Measure the variant a phone will actually download, not the universal IPA.

Bitcode is not a thinning tool you can still use. Starting with Xcode 14, Apple no longer accepts bitcode submissions. The build-metadata page says dSYM downloads for bitcode exist only for older submissions. Do not enable bitcode to shrink a new app.

### On-Demand Resources and Background Assets

On-Demand Resources are deprecated as of iOS 27, iPadOS 27, tvOS 27, and visionOS 27, and support will be removed in future releases. Starting in early 2027, App Store Connect will no longer accept uploads that contain On-Demand Resources from new apps or from apps that have never used the feature. Migrate to Background Assets. Source: [On-Demand Resources size limits](https://developer.apple.com/help/app-store-connect/reference/app-uploads/on-demand-resources-size-limits/).

Do not introduce On-Demand Resources in a new app. [Background Assets](https://developer.apple.com/documentation/backgroundassets) can download extra files before first launch, in the background, or on demand, and assets can update without a new app build. If you use Apple-hosted Background Assets, the product page shows an “Up to” size that includes non-localized essential packs and the largest single language of localized essential packs ([Maximum build file sizes](https://developer.apple.com/help/app-store-connect/reference/app-uploads/maximum-build-file-sizes/)).

For the price-tag scanner, do not put text recognition or the trip ledger in a download that can be absent at first launch. The core flow has to work in a store with no signal. See `offline-and-scanning.md`.

## Google Play

New apps have had to publish an Android App Bundle since August 2021. Source: [About Android App Bundles](https://developer.android.com/guide/app-bundle). The same banner appears on [APK Expansion Files](https://developer.android.com/google/play/expansion-files).

All Play size limits below are **compressed download size**, computed by Play Console on upload. `bundletool` is a close estimate, not the same number. Source: [Play app size limits](https://support.google.com/googleplay/android-developer/answer/9859372?hl=en).

| Component | Compressed download limit | Kind |
| --- | --- | --- |
| Base module | 500 MB | Store limit |
| Each feature module | 500 MB | Store limit |
| Each asset pack | 1.5 GB | Store limit |
| All modules plus install-time asset packs, together | 4 GB | Store limit |
| Asset packs delivered on demand or fast-follow, together | 30 GB | Store limit |
| Total of those caps | 34 GB | Store limit |

Play’s own note on that page: keep the app as small as possible, and most apps should stay well below these limits.

Above **200 MB**, a person installing on mobile data sees a non-blocking dialog that the app is large. That is a store warning, not a failed upload. Source: the same size-limits page.

### What 200 MB is, and what it is not

A request that says “about 200 MB initial download, unless you add expansion files” is describing an older shape of the rules. State the current numbers instead.

- The current hard cap on the base module is **500 MB** compressed, not 200 MB.
- The current cap on the install (modules plus install-time asset packs) is **4 GB** compressed.
- **200 MB** is the size at which Play shows a non-blocking mobile-data dialog.
- The app-bundle page and the expansion-files page both say that new apps larger than 200 MB are supported by Play Feature Delivery or Play Asset Delivery. That sentence records when those features became the way to ship a large app. It is not a current 200 MB hard cap.
- Apps that still publish APKs, instead of bundles, are on the legacy APK cap: **100 MB** compressed. That is a different, older path. Source: the size-limits page.

[About Android App Bundles](https://developer.android.com/guide/app-bundle) also says the compressed APKs required to install (base plus configuration splits) must be no more than 4 GB, and that any later on-demand feature-module download must meet that restriction too. It adds that asset packs do not contribute to that APK-set limit and have their own limits. The Play Console table is more specific: **install-time** asset packs count toward the 4 GB cumulative total. When you are deciding whether an upload will be rejected, use the Play Console table. Re-read both pages if they diverge further.

### Asset delivery, feature delivery, and OBB

[Play Asset Delivery](https://developer.android.com/guide/playcore/asset-delivery) is how a bundle carries large assets. It describes itself as the replacement for legacy expansion files (OBB) for games larger than 200 MB.

| Mode | When it arrives | Counts toward the size listed on Play | May be missing later |
| --- | --- | --- | --- |
| Install-time | With the install, as split APKs | Yes | No. The person cannot delete these. |
| Fast-follow | Right after install, without opening the app | No | Yes. Play or the person may remove them. |
| On-demand | While the app is running | No | Yes. Same as fast-follow. |

Install-time packs also count toward the 4 GB cumulative limit in the table above. Fast-follow and on-demand packs use the separate 30 GB cap. Do not put anything the first screen needs into fast-follow or on-demand. The price-tag scanner’s recognizer belongs in the base module.

[Play Feature Delivery](https://developer.android.com/guide/playcore/feature-delivery) is the same idea for code modules. A module the first scan needs is not an on-demand module.

App bundles do not support APK expansion (`*.obb`) files. Source: [About Android App Bundles](https://developer.android.com/guide/app-bundle). The expansion-files guide still documents the old APK path (a compressed APK of at most 100 MB, plus up to two expansion files of 2 GB each) and then tells you to use a bundle, which allows up to a 500 MB compressed download, instead of expansion files. Treat OBB as legacy. Do not add it to a new app.

## Price-tag scanner

The scanner’s download should be the base install: app code, the on-device recognizer, and any small offline table (for example a tax-rate table). No expansion file, no On-Demand Resources, no fast-follow pack on the critical path. Hold the line at the budgets in `budgets.md`, which sit well under 100 MB and under the 200 MB warning on both stores.

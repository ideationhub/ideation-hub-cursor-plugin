# Size and startup budgets

Read this when you need a number to aim at, or a way to measure the binary and the launch. Store caps are in `size-limits.md`. A budget is tighter than a cap, and you write it down before the binary grows.

## Set the budget

1. Write two download budgets, one per store, in compressed or thinned download size. Also write an install-size budget if disk use matters.
2. Write startup budgets for cold, warm, and hot start. Say which device class they apply to (a mid-range phone several years old, not the newest flagship).
3. A budget is a fail in your own check, not a store rejection. Say that when you report a measurement.
4. Leave headroom. Apple’s processed size can be larger than the IPA you uploaded, because of DRM and recompression. Play’s `bundletool` estimate is not identical to Play Console.

For a small utility, target **well under 100 MB** on both stores. That keeps you under the 200 MB warning on each store (`size-limits.md`) with room for one or two dependencies.

### Worked example: the price-tag scanner

| Budget | Value | Why this number |
| --- | --- | --- |
| iOS thinned download | ≤ 30 MB | A native or thin client plus Apple’s built-in text recognizer, which adds no model. |
| Android compressed download | ≤ 25 MB | The same app plus a bundled Latin text model, about 4 MB per CPU architecture (`offline-and-scanning.md`). |
| Cold start, mid-range phone | under 2 seconds | Inside the Android vitals “slow” line, with margin. |
| Warm start | under 1 second | The vitals “slow” line is 2 seconds. |
| Hot start | under 0.5 seconds | The vitals “slow” line is 1 second. |

These are example budgets for this kind of app. Raise them only for a feature the app cannot ship without, and record what you refused to add. Do not raise them to make room for a second copy of a recognizer, an unused cross-platform runtime, or an asset pack on the first screen.

Apple does not publish a startup number on the pages this skill uses. Do not invent one and attribute it to Apple. The cold-start budget above is ours. The vitals lines below are Google’s.

## Measure download size

### iOS

Do not treat the `.app`, the `.xcarchive`, or the IPA you upload as the size a person downloads. Those bundles contain files the App Store does not ship, including dSYMs. Source: [Reducing your app’s size](https://developer.apple.com/documentation/xcode/reducing-your-app-s-size).

During development, export an Ad Hoc, Development, or Enterprise archive with app thinning set to all compatible device variants. The export folder contains `App Thinning Size Report.txt`. Compressed size in that report is the download. Uncompressed size is the install. The same page shows how to produce the report from `xcodebuild -exportArchive` with a `thinning` key.

After upload, the accurate numbers are in App Store Connect: TestFlight, the build, Build Metadata, App File Sizes. Read **download size** and **install size** per device variant. A yellow warning means a variant crossed the 200 MB over-the-air download limit. TestFlight builds are larger than the App Store build. The approved binary can still grow slightly. Source: [View builds and metadata](https://developer.apple.com/help/app-store-connect/manage-builds/view-builds-and-metadata/) and the reducing-size page.

Compare the variant for a current phone with the iOS budget. The universal variant is not the number a current phone downloads.

### Android

Play Console computes compressed download size when you upload. Before upload, `bundletool` is the estimate, and it is not identical. Source: [Play app size limits](https://support.google.com/googleplay/android-developer/answer/9859372?hl=en).

After a production release, read Monitor and improve > Android vitals > App size. That page shows download size on a reference device (`XXXHDPI ARMv8`, or the closest configuration you support) and the range across configurations. The same page breaks the size into code/DEX, resources, assets, native libraries, and other, for the last few releases.

To see what Play generated from a bundle, open Test and release > Latest releases and bundles > All app versions > App bundle explorer, then the Delivery tab. The Modules table includes the base module and its download size. Asset packs appear when you have them. The help article that describes this path is [Inspect app versions on the Latest releases and bundles page](https://support.google.com/googleplay/android-developer/answer/9844279?hl=en). On 26 November 2026 a request for that article returned HTTP 404 while still serving a page with that title. If the link fails, use the console path above. [Build and test your Android App Bundle](https://developer.android.com/guide/app-bundle/test) is the developer page for inspecting generated APKs.

Compare the reference-device download with the Android budget, and check the top of the range so one ABI or density does not blow the budget alone. Bundled native code is per architecture. A fat APK is not what Play serves from a bundle. Measure the split the device gets.

## Measure startup

### Android

[App startup time](https://developer.android.com/topic/performance/vitals/launch-time) defines the three starts. A cold start creates the process. A warm start redoes some of that work. A hot start brings an activity that is still in memory to the foreground. Optimize for cold start. Time to initial display is the time until the first frame, including a splash screen. Call `reportFullyDrawn` when the app is actually usable, or a splash will hide a slow start.

Play records this in Android vitals and flags a session as slow at:

| Start | Slow at |
| --- | --- |
| Cold | 5 seconds or more |
| Warm | 2 seconds or more |
| Hot | 1 second or more |

Source: [Android vitals](https://support.google.com/googleplay/android-developer/answer/9844486?hl=en), section “Start-up and loading times”. Vitals records the day’s maximum when the app starts more than once from the same state, and only when a person launches an activity.

Treat those lines as the store’s bad-behavior marks, not as a target. The worked-example budgets sit under them. A 5 second cold start is already a slow session.

### iOS

Read launch metrics from shipping builds in Xcode Organizer ([Analyzing the performance of your shipping app](https://developer.apple.com/documentation/xcode/analyzing-the-performance-of-your-shipping-app)) and from [MetricKit](https://developer.apple.com/documentation/metrickit). The Apple pages this skill uses do not publish a numeric “slow launch” threshold. Report the measurement and the budget. Do not cite a fake Apple limit.

## When a measurement misses the budget

Say what the number was, which variant or device, and which budget it missed. Name the largest slice from the Play breakdown or the thinning report. Cut that slice, or move it off the install path using the rules in `size-limits.md`, before you raise the budget. Do not “fix” a miss by switching the recognizer to a download the first launch depends on. That breaks `offline-and-scanning.md`.

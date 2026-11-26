# Offline use and on-device scanning

Read this when the app must work without a network, or when it reads text from the camera. The price-tag scanner is the example. The same split applies to any app whose main task happens somewhere the network is poor.

## Offline-first

The core flow does not wait on the network. For the scanner that flow is: open the camera or accept a typed price, read the tag, choose the price, add the line, show the trip total, and keep the trip if the app is killed.

Put that state in local storage on the device (a database or files in the app’s own storage). The credit balance the UI shows is a cache of the ledger, not a request. Buying a pack, proving the paid download, and recovering credits after a reinstall are the calls that need a network. Queue them and run them when a connection exists. Do not block the open trip on that queue.

If a download can be deleted or can fail, the core flow must not need it.

- Apple On-Demand Resources are deprecated for new apps (`size-limits.md`). Do not put the recognizer there.
- Google Play fast-follow and on-demand asset packs may be removed, and the app must not assume they exist ([Play Asset Delivery](https://developer.android.com/guide/playcore/asset-delivery)). Install-time content is the only Play-delivered content that is present at first launch.
- An unbundled ML Kit model downloads through Google Play services the first time it is used, unless you declare an install-time download. Either way, the first scan can fail with no signal. See below.

A cloud recognizer is a fallback for a photo the on-device model cannot read, and only while online. It is not the primary reader. Sending the photo off the device is also a privacy disclosure (`privacy-and-permissions.md`). The default for the scanner is that the tag image stays on the phone.

Manual entry is part of the core flow, not an error state. The person can type the price when the camera is denied, the model is missing, or the tag is unreadable.

## Text recognition

Pick one on-device recognizer per platform. Do not ship two for the same script.

| Option | Where it runs | Added download | Works offline from install | Use when |
| --- | --- | --- | --- | --- |
| Apple Vision text recognition | On device, in the OS | 0 MB | Yes | iOS. This is the default for the scanner. |
| ML Kit text recognition v2, bundled | On device, linked into the app | About 4 MB per script per CPU architecture | Yes | Android, when the first scan must work with no network. |
| ML Kit text recognition v2, unbundled | On device, model from Play services | About 260 KB per script architecture, then a model download | Not until the model has downloaded | Android, only if you can tolerate a first-run download and you still have a typed-price fallback. |

Sources: [Recognizing text in images](https://developer.apple.com/documentation/vision/recognizing-text-in-images) and [ML Kit text recognition v2 on Android](https://developers.google.com/ml-kit/vision/text-recognition/v2/android). The ML Kit page states the size and the install difference in its comparison table. Bundled Latin is `com.google.mlkit:text-recognition`. Unbundled Latin is `com.google.android.gms:play-services-mlkit-text-recognition`. Read that page for the current artifact versions. Do not copy a version out of this skill into a build file without checking the page.

On iOS, call Vision (`VNRecognizeTextRequest`) on a still frame or a sample buffer. Do not add ML Kit on iOS for the same job. The Apple API is already on the device.

On Android, for the scanner, bundle the scripts you actually read. Latin is the usual shelf-tag script in many markets. Each extra bundled script costs about another 4 MB per architecture, so an unnecessary script is a budget failure (`budgets.md`). If you use the unbundled library, declare the install-time model download in the manifest (`com.google.mlkit.vision.DEPENDENCIES`) or request it through Play services, and still let the person type the price when the model is not there yet. The ML Kit page says requests made before the download finishes produce no results.

Recognition quality depends on pixels on the character, focus, and how much of the frame the tag fills. The ML Kit page asks for at least about 16×16 pixels per character and says there is little gain past about 24×24. For a live scanner, drop frames while a recognition is in flight. CameraX’s default `ImageAnalysis` backpressure (`STRATEGY_KEEP_ONLY_LATEST`) does that. Close the `ImageProxy` when the task finishes.

The scanner reads prices on the one tag in front of the camera. It does not need a catalog, a network price API, or a model of every product.

## Camera

### iOS

Use AVFoundation when you need your own capture pipeline (session, device, photo or video output) and Vision on the frames. [AVFoundation](https://developer.apple.com/documentation/avfoundation) is the capture stack.

Use VisionKit when a system scanning UI is enough. [Scanning data with the camera](https://developer.apple.com/documentation/visionkit/scanning-data-with-the-camera) covers `DataScannerViewController`: live video, highlighting, and text and codes. Check `isSupported` (A12 and later) and `isAvailable` (camera permission and no other restriction) before presenting it. [DataScannerViewController](https://developer.apple.com/documentation/visionkit/datascannerviewcontroller) is the class reference.

`NSCameraUsageDescription` is required for either path. Wording is in `privacy-and-permissions.md`. If the key is missing, the camera API fails at runtime.

Guideline 2.5.14: the person gives explicit consent, and the app shows a clear indication when the camera is recording. Guideline 4.10: you may not charge for the camera itself. The pack of trip credits buys tallies, not camera access. Source: [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) (last updated 16 October 2026 on the page read for this skill).

### Android

Use [CameraX](https://developer.android.com/media/camera/camerax) (`ImageAnalysis`) and pass `ImageProxy` frames to ML Kit. Request the camera permission in context, at the moment the person starts a scan, not on a cold launch that is only showing a home screen. Runtime permissions are in `privacy-and-permissions.md`.

If the person denies the camera, both platforms stay usable: the tally accepts a typed price. Do not loop the permission prompt. Do not disable the rest of the app.

# Privacy and permissions

Read this when the app uses the camera or location, or when you are filling in the store privacy forms. The price-tag scanner uses the camera to read a tag. It uses location only if a feature needs it (for example a local tax rate). Both have to be optional for the core tally: a typed price and a typed place still work.

Never print a credential, a device check token, or a purchase token while you are testing these flows.

## Camera wording

### Apple

[`NSCameraUsageDescription`](https://developer.apple.com/documentation/bundleresources/information-property-list/nscamerausagedescription) is required if the app uses a camera API. The system shows your string in the permission dialog. Missing key: the camera API fails at runtime. [Scanning data with the camera](https://developer.apple.com/documentation/visionkit/scanning-data-with-the-camera) walks through adding the key and says the dialog includes the app name and your reason.

Guideline 5.1.1(ii): purpose strings clearly and completely describe the use, and paid functionality must not depend on granting access to the data. Guideline 5.1.1(iv): respect a denial, and offer an alternative where you can. Guideline 2.5.14: explicit consent, and a clear indication while the camera is recording. Source: [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/).

Write the reason as what the person gets, not as the framework name. For the scanner: “The camera reads the price printed on the shelf tag. The picture stays on this phone.” Do not say “for OCR” or “to improve your experience.” If a later version uploads a failed tag, change the string and the privacy label first. The string and the behavior have to match.

### Android

Request `CAMERA` at runtime, in context, when the person starts a scan. [Request app permissions](https://developer.android.com/training/permissions/requesting) is the runtime-permission guide. Declare `android.permission.CAMERA` in the manifest. If they deny it, open typed entry. Do not send them to settings on the first denial.

[Permissions policy](https://support.google.com/googleplay/android-developer/answer/16909972?hl=en) (the page is titled as a preview of upcoming changes and also contains the current policy): request sensitive permissions in context, only for a feature the listing promotes, and provide an alternative when the person declines. The camera is a restricted permission. Use it for scanning tags, not for anything the listing does not describe.

## Location

Ask only if a feature needs it, and ask at the moment that feature runs, not at launch.

Apple: [`NSLocationWhenInUseUsageDescription`](https://developer.apple.com/documentation/bundleresources/information-property-list/nslocationwheninuseusagedescription). Prefer When In Use. Guideline 5.1.5: location only when it is relevant to a feature, and explain why. Guideline 5.1.1(iv) uses this example: if the person declines location, let them type an address. For a tax rate, that alternative is a typed postal code. [Requesting authorization to use location services](https://developer.apple.com/documentation/corelocation/requesting-authorization-to-use-location-services) is the API page.

Google: foreground location is `ACCESS_COARSE_LOCATION`, `ACCESS_FINE_LOCATION`, or both. Coarse is an area of about 3 square kilometers. Precise is on the order of 50 meters or better. Request the coarser one that still does the job. A postal code usually does not need precise location. Source: [Request location permissions](https://developer.android.com/develop/sensors-and-location/location/permissions).

### Location button from 27 January 2027

Two official texts, both checked on 26 November 2026:

- The developer guide says: if the app targets Android 17 (API 37) or later and only contains features that need location for one session, Play policy requires the location button. Source: [Request location permissions](https://developer.android.com/develop/sensors-and-location/location/permissions), “One-time access”.
- The Play policy page is explicitly a **policy preview**. Its opening summary says, effective **January 27, 2027**: apps targeting Android 17 (API 37+) must use the Android location button for one-time or infrequent location requests, and using `ACCESS_FINE_LOCATION` requires a Play Console declaration of why coarse location or the location button is not enough. The same page’s location section says that, for apps targeting SDK 37, precise location for a one-time user-initiated action must use the location button and declare `onlyForLocationButton`, and that you may request `ACCESS_FINE_LOCATION` only when neither the location button nor coarse location can support the feature. The page says the full policy wins if a summary conflicts with it. Source: [Preview: Permissions and APIs that Access Sensitive Information](https://support.google.com/googleplay/android-developer/answer/16909972?hl=en).

For the scanner, a tax-rate lookup is one-off. Plan on the location button when you target Android 17, prefer coarse location, and keep typed postal-code entry. Do not request background location for this.

## What counts as collection

Data that never leaves the phone is treated differently from data you send to a server. The tag image, the on-device recognition, and a location used only to pick a rate on the phone are in the first group if you do not upload them.

Apple privacy nutrition labels: “Data that is processed only on device is not ‘collected’ and does not need to be disclosed.” If you derive something and send that off the device, judge the derived data on its own. Payment card data entered in Apple’s sheet, which you never receive, is not collected. Source: [App privacy details](https://developer.apple.com/app-store/app-privacy-details/). You still submit the label. Every app needs a privacy policy link in App Store Connect and inside the app (guideline 5.1.1(i)).

Google Play Data safety: the form is required even when you collect nothing, and it needs a privacy policy link. “User data accessed by your app that is only processed locally on the user’s device and not sent off device does not need to be disclosed.” Pseudonymous data that can be re-associated with a person must be disclosed. Ephemeral processing that does leave the device has its own definition on that page. You are responsible for the answers. Source: [Data safety](https://support.google.com/googleplay/android-developer/answer/10787469?hl=en).

A credit-recovery backend that stores an `appTransactionID` or a Play account key is collection if it leaves the device. Declare it. Do not describe that identifier as “no data collected.”

## Apple privacy manifest

Ship a `PrivacyInfo.xcprivacy` in the app, and expect one inside each third-party SDK that collects data or uses a required-reason API. The manifest lists collected data types and the required-reason APIs. Source: [Privacy manifest files](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files).

Required-reason APIs are APIs that can fingerprint a device. [Describing use of required reason API](https://developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api) says App Store Connect does not accept an app that uses those APIs without a declared reason, starting 1 May 2024. The reason has to match the real use. A third-party SDK declares its own reasons. You cannot cover the SDK’s use from the app manifest. Check plugins and analytics SDKs before you upload. A scanner that only uses the camera, Vision, and local storage may still pull a required-reason API in through a dependency. Look at the privacy report Xcode generates. Do not paste a reason code you have not matched to the API list on that page.

## Accounts

Guideline 5.1.1(v): if the app has no significant account features, it works without a login. If the app supports creating an account, it offers account deletion inside the app. Do not require personal information unless it is core to the app or required by law.

The scanner does not need a name or an email to scan a tag. Prefer `appTransactionID` on iOS over an account (`purchases.md`). If you do create an account on Android because Play does not give you an equivalent id, deletion has to exist in the app, and the Data safety form has to match what you store. The Play help URL `answer/13327111` (account deletion), cited in older notes, returned HTTP 404 on 26 November 2026. Read the current Play policy in Console before you rely on a specific deletion URL.

## Price-tag scanner, minimum disclosures

| Data | Leaves the phone? | Label |
| --- | --- | --- |
| Camera frames used for on-device recognition, then discarded | No | Not collected, on both stores, while this stays true |
| Trip lines and totals stored only on device | No | Not collected |
| Credit ledger on your server, keyed by a store id | Yes | Declare it |
| Precise or coarse location sent to a server | Yes | Declare it. Prefer keeping the lookup on device. |
| Card data entered in the store’s payment sheet | You never receive it | Not collected by you |

Say this in the purpose strings and in the forms. If the implementation changes, change the forms in the same release.

# Purchases, credits, and refunds

Read this when the app sells a digital good, especially a consumable the person can buy again. The worked example is trip credits for the price-tag scanner: the paid download may include a starter set, and further packs are consumable in-app purchases.

A paid download and an in-app purchase are different products. Do not implement the starter credits as a consumable the app buys from itself at install.

## Which store API

Digital goods inside the app go through the store.

- Apple: [StoreKit](https://developer.apple.com/documentation/storekit). Use StoreKit 2. Guideline 3.1.1: unlocking functionality uses in-app purchase. Source: [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/).
- Google Play: [Google Play Billing](https://developer.android.com/google/play/billing) and [Integrate the Google Play Billing Library](https://developer.android.com/google/play/billing/integrate). [Payments policy](https://support.google.com/googleplay/android-developer/answer/10281818?hl=en): Play’s billing system is required for in-app digital goods (virtual currencies, extra playtime, additional features) unless that policy permits something else. The policy documents alternative billing in some regions. Read it before you add a second checkout. The older help URL `answer/9858738` returned HTTP 404 on 26 November 2026. Do not cite it.

Guideline 2.3.2: the description, screenshots, and previews must show which items need an extra purchase. For the scanner, the listing says the download includes the starter credits and that further packs are bought in the app.

Do not charge for the camera or another built-in capability (Apple guideline 4.10). The product is the credits.

## Consumable credits

A pack of trip credits is a consumable: the person can buy it, use it, and buy it again.

Apple guideline 3.1.1: “Any credits or in-game currencies purchased via in-app purchase may not expire,” and restorable purchases need a restore mechanism. Do not put an expiry date on a credit. Do not delete unused credits because a trip ended or a month passed.

Consumables do not appear in [`Transaction.currentEntitlements`](https://developer.apple.com/documentation/storekit/transaction/currententitlements). That sequence is non-consumables, active subscriptions, and non-renewing subscriptions. Apple will not hand the remaining credit balance back. Unfinished consumables are on `Transaction.unfinished` and `Transaction.all`. Finish the transaction only after the credits are durably granted.

On Play, a consumable is a one-time product you consume after granting it, so it can be bought again. If your app has a backend, consume with `Purchases.products:consume` after you have checked `consumptionState`. A client-only app uses `consumeAsync`. Consuming acknowledges the purchase. Source: the integrate guide linked above.

### Grant, then acknowledge, within three days

Play refunds a purchase you do not acknowledge. The integrate guide says to acknowledge within three days of the purchase, or Play refunds it and revokes the entitlement. The three-day window starts when the state becomes `PURCHASED`, not while it is `PENDING`. Do not acknowledge a pending purchase.

Order of operations:

1. The store reports a purchase (`PURCHASED` / a verified StoreKit transaction).
2. Your backend verifies it with the store (Play Developer API `Purchases.products:get`, or Apple’s signed transaction). Reject duplicates. On Play, `purchaseToken` is the unique key. Do not use `orderId` as the primary key. Promo-code purchases may have no `orderId`. Source: [Fight fraud and abuse](https://developer.android.com/google/play/billing/security).
3. Add the credits to your ledger once.
4. Then acknowledge or consume. On Apple, finish the transaction.

The Billing Library version shown on the integrate page when this skill was written was 9.1.0. Read that page for the version to depend on. Do not freeze this sentence into a build file.

## Proving the download was paid

The starter credits come with the paid app, so you need proof of the app purchase, separate from a pack purchase.

### Apple

[`AppTransaction`](https://developer.apple.com/documentation/storekit/apptransaction) is the signed app purchase. It requires iOS 16, iPadOS 16, or the matching later OS on other platforms. [`appTransactionID`](https://developer.apple.com/documentation/storekit/apptransaction/apptransactionid) is a single id per Apple Account for that app. It stays the same if the person redownloads on any device, gets a refund and buys again, or changes storefront. It exists even when there are no in-app purchases. With Family Sharing, each family member has their own id.

Use `appTransactionID` as the account key for the starter credits if you are not asking the person to create an account. Check `revocationDate` before you treat the purchase as still valid. A reinstall of a paid app is free. Without a ledger of how many starter credits were used, a reinstall looks like a new grant. Record the grant against `appTransactionID` on your backend the first time you see that id.

### Google Play

[Play Integrity verdicts](https://developer.android.com/google/play/integrity/verdicts) expose `accountDetails.appLicensingVerdict` for the account signed in on the device: `LICENSED`, `UNLICENSED`, or `UNEVALUATED`. `LICENSED` means that account installed or bought the app from Play. It is not a stable per-person purchase id. The verdict page notes that on some older devices the account can stay licensed after uninstall. `UNEVALUATED` means Play did not assess licensing (untrusted device, unknown app version, or not signed in).

The older [Licensing Verification Library](https://developer.android.com/google/play/licensing) also answers whether the user is licensed. It does not give you an id equivalent to `appTransactionID`.

So a paid Android app can tell you “this Play account is licensed” and cannot, from the pages above, give you a durable id that survives reinstall without your own record. If you need to grant starter credits once, you need a backend key you choose, and you have to be honest about what identifying data that key is. Creating an account, including a silent guest account, has store consequences in `privacy-and-permissions.md`.

### Paid and free are not symmetric on Play

[Set up your app’s prices](https://support.google.com/googleplay/android-developer/answer/6334373?hl=en): you can change a paid app to free. Once the app has been offered for free, it cannot be changed to paid. Charging later means a new app with a new package name. Decide the paid download before the first production release if you might want it later.

## Restoring credits

Neither store restores a consumable balance.

- Apple’s restore mechanism is for restorable products (non-consumables and subscriptions), which is what guideline 3.1.1’s “restore mechanism” refers to. Consumables are absent from `currentEntitlements`.
- Play will deliver an unconsumed purchase again through `queryPurchasesAsync` until you consume it. After a successful consume, the remaining credit count lives only in your ledger.

Recovering unused trip credits after a reinstall or a new phone is your backend: the Apple `appTransactionID`, or whatever Android key you established, maps to a balance. The phone caches that balance for offline use and reconciles when it is online. A lost phone with tallies that were never synced can overspend. Say that plainly when you design the cache. Do not pretend StoreKit or Play Billing will reconstruct the balance.

## Refunds and revocations

Your own “no refunds” copy does not stop the store. Apple and Google can refund a paid app or a pack. Reconcile the ledger when they do.

- Apple: `revocationDate` on `AppTransaction` for the app purchase, and revocation on the in-app transaction. [App Store Server Notifications](https://developer.apple.com/documentation/appstoreservernotifications) deliver the refund and revoke events to your server. Do not rely on the phone being online to learn about a refund.
- Google: [Voided Purchases API](https://developers.google.com/android-publisher/voided-purchases). [Fight fraud and abuse](https://developer.android.com/google/play/billing/security) says a voided purchase includes cancellations, chargebacks, and developer refunds that revoke, and that unacknowledged purchases which Play auto-cancels are in that set. Claw back credits that were not used. If they were already used, the same page’s example is to allow a negative balance and to limit further purchases until it is positive. Pick one policy and apply it the same way on both stores. Do not leave the person in an active trip with a silent negative balance and no explanation.

Verify on a server before you grant. A client that only trusts a local flag will grant the starter credits again after a reinstall, and will keep credits after a refund.

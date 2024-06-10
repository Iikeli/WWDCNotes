# What’s new in StoreKit 2 and StoreKit Testing in Xcode

Get to know the latest enhancements to StoreKit 2 and StoreKit Testing in Xcode. Discover API updates for promoted in-app purchases, StoreKit messages, the Transaction model, the RenewalInfo model, and the App Store sheet for managing subscriptions. Learn how to upgrade to SHA-256 for on-device receipt validation and use APIs to create SwiftUI views.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10140", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## API Updates
- Promoting of In App Purchases possible in Connect
- Receive promotion purchases with `PurchaseIntent.intents`
- Need to call `.purchase` after checking `.canProcessPurchase`
- You can chang eorder of promotions locally
- You can also change visibility in App Store
- New fields `.storefront`, `.storeFrontCountryCode`, and `reason` (customer purchase vs. recurring) field in `Transaction`
- New field `renewalDate` on `RenewalInfo`
- New fields above are back-deployed to iOS 15!
- New message reason "billingIssue" added in Messages API
- Signing receipts migrates from SHA1 to SHA256 starting June 20, migration complete by August 24
- If you're using StoreKit, nothing to do, StoreKit uses SHA256 already

## Build SwiftUI apps
- New `ProductView` for showing exactly 1 `productID` (like non-consumable)
- New `StoreView` for showing many ids in a list
- New `SubsriptionStoreView` for showing all subsriptions in a subscription group in a list
- Many customization options for these views
- Use `.manageSubscriptionSheet(isPresented:groupID:)

## StoreKit Testing in Xcode
- Transaction manager shows all apps and devices now
- Open manager via "Debug" menu in Xcode
- Purchases can be created directly on the Mac in transaction manager via + button
- Lots of customizability when creating purchases, supoprts offers/promotions/reneweable or not/purchase date etc.
- No debug session required
- New "Configuration Settings" in StoreKit configuration file: Change Storefront/Localization, pass Options, simulate Failures
- Set also a "renewal rate" with from 15 minutes down to just 2 seconds
- `SKTestSession` with purchase APIs allows the same adjustments as listed above in unit tests via `.setSimulatedError`, `.timeRate`, etc.

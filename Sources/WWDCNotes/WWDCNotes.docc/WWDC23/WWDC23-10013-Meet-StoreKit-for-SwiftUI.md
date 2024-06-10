# Meet StoreKit for SwiftUI

Discover how you can use App Store product metadata and Xcode Previews to add in-app purchases to your app with just a few lines of code. Explore a new collection of UI components in StoreKit and learn how you can easily merchandise your products, present subscriptions in a way that helps users make informed decisions, and more.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10013", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



![][EncapsulatedLogic]

[EncapsulatedLogic]: EncapsulatedLogic.png

- Assets, Localization, Style provided by the Developer
- `StoreView`, `ProductView`, and `SubscriptionStoreView` can be used when `StoreKit` and `SwiftUI` are imported
- `StoreView(ids: ["..."]) { product in /* custom icon view */ }` presents a full screen view with a list if `ProductView`s
- Define your custom layout by using `ProductView` directly
- Customize with `.productViewStyle(.large|.compact|.regular)`, `.containsBackground`, `.backgroundStyle(.clear)` & more
- Customize with `.subscriptionStoreButtonLabel(.multiline)`, `.sbuscriptionStorePurchaseItemBackground(.thinMaterial)` & more
- New `.onInAppPurchaseCompletion { product, result in ... }` modifier for SwiftUI (catching purchases in descending views)
- Use `.onInAppPurchaseStart { product in ... }` for showing progress indicator
- Use `.subscriptionStatusTask(groupID: ...) { taskState in ... }` with state cases `.loading`, `.failure(any Error)`, and `.success(Transaction)`
- Or `.currentEntitlementTask(for ...) { state in ... }` for non-consumable products
- Customize with `.productIconBorder` if you want the App Store product icon look
- Create custom styles by conforming to `ProductViewStyle`
- Use `.storeProductsTask(for: ...)` modifier in custom styles to use Apple-provided caching & loading logic
- Customize with `.storeButton(.visibble|.automatic|.hidden, for: .redeemCode|.cancellation|.restorePurchase|.signIn|.policies)` to control buttons shown

![][AuxiliaryButtons]

[AuxiliaryButtons]: AuxiliaryButtons.png

![][RedeemCode]

[RedeemCode]: RedeemCode.png

![][Policies]

[Policies]: Policies.png

- New `.subscriptionStoreSignInAction { ... }` modifier triggered on press of `.signIn` store button
- `.policies` shows terms and privacy policy buttons above plan list (in iOS / macOS)
- Customize with `.subscriptionStorePolicyForegroundStyle(.white)` for legibility on custom backgrounds
- Also `.subscriptionStoreControlStyle(.buttons|automatic|.picker|.prominentPicker)` available for pre-defined styles

![][ButtonsStyle]

[ButtonsStyle]: ButtonsStyle.png

- Use `.subscriptionSToreButtonLabel(.displayName|.price)` in custom style
- `.subscriptionStoreControlIcon { subscription, info in ... }` for further styling
- Use `.containerBackground(.accent.gradient, for: .subscriptionStore|Header|FullHeight)` options for background
- Use `visibleRelationships: .upgrade` parameter on initializer to change to "Upgrade" mode when already subscribed

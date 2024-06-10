# What’s new with in-app purchase

Create a great in-app purchase experience for your iPhone, iPad, Mac, and Apple Watch apps. Discover how to handle refunds, integrate new App Store server notifications, and find out how to use receipts and server notifications to manage subscriber status. We’ll also walk you through the latest updates in StoreKit, including in-app purchases on Apple Watch, Family Sharing, SKOverlay, SKAdNetwork, and more.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10661", purpose: link, label: "Watch Video (45 min)")

   @Contributors {
      @GitHubUser(javierdemartin)
   }
}



## Introduction

Apps running locally from Xcode and distributed it through TestFligt use the Sandbox environment. Once the app is in the App Store it will use the production environment. Testing app purchases can become cumbersome but Xcode introduces StoreKit testing to ease these steps.

## StoreKit Testing

With StoreKit testing you will be able to test your app's in-app purchases entirely locally. This is done via a new framework, [StoreKit Test](https://developer.apple.com/documentation/storekittest#).

**By default, StoreKit uses Sandbox environment**, any testing in-app purchase you create will not appear until you either enable local testing within Xcode or create those in-app purchases inside App Store Connect.

To enable local testing you need to define those in-app products inside your project. To do so, create a new file of type **StoreKit Configuration File** template. This new file allows you to define all the necessary metadata for your in-app products. There are three types of in-app purchases you can create from the bottom left side of the screen:

1. **Consumable** In-App Purchase: You can buy it over and over again (lives or gems within a game).
2. **Non-Consumable** In-App Purchase: Purchased once and never expire.
3. **Auto-Renewable Subscription**: Users are charged periodically for access to services or content.

Once your StoreKit Configuration is ready it's time to tell Xcode to use this configuration instead of the Sandbox environment when launching your app. This is done under "Edit Scheme" inside the "Scheme Editor" and under "Run Options" select the StoreKit Configuration you would like to use. Now when you relaunch your app it will use that environment.

Every change you do to your `.storekit` file is rendered automatically on your device. No need to recompile your app to see these changes.

Whenever you want to test the flow and actions of purchasing an item there's no need to log into a Sandbox account, it will use your local testing environment if selected. It will update your app's payment transaction observer just like it did on your sandbox or production environment. **You will also receive a receipt that your app can verify when using local StoreKit testing**. To modify or remove StoreKit transactions you can use the new StoreKit Transaction Manager, you can also simulate a refund that will update the receipt to contain a cancellation date for when the refund occurred. This new framework also supports Ask to Buy to test kid's permissions. This is enabled under the "Editor" menu and selecting "Enable Ask to Buy" on any `*.storekit` file this will be reflected on the StoreKit Transaction Manager and the StoreKit Transaction Observer will mark it to be in the [deferred](https://developer.apple.com/documentation/storekit/skpaymenttransactionstate/deferred#) state until it's approved.

Auto Renewable Subscriptions is also supported and allows to test promotional offers and introductory offers. To reduce waiting times until these offers expire you can modify how much they're going to last. Under the "Editor" menu, select "Time Rate" to modify the time scale to test your app's renewal statuses.

A new API [`didRevokeEntitlementsForProductIdentifiers`](https://developer.apple.com/documentation/storekit/skpaymenttransactionobserver/3564804-paymentqueue#), launched in iOS 14, allows your app to tell a user it is no longer entitled to one or more in-app purchases.

## Receipt Validation with local Xcode testing environment

Every app transaction is reflected on the **app's receipt**. This is a **signed proof from the App Store of every record of purchases made by the user in your app**. It's **stored on the device and it's updated automatically by the system**. It is signed so you know it came from the App Store and was meant for your app on that device.

There are some key differences when working with the local Xcode StoreKit testing environment and the Sandbox environment for the app's receipt.

1. They are signed with a different private key than what is used for the receipts generated in the Sandbox or production environments. Hence the need to use a different certificate when validating. Said certificate is exportable from the StoreKit configuration editor menu.
2. **The StoreKit Test Certificate is not part of a certificate chain**. Apple recommends defining a debug macro to differentiate both environments within your client side verification code.

```swift
#if DEBUG
let certificateName = "StoreKitTestCertificate
#else
let certificateName = "AppleIncRootCertificate"
#endif

// Verify Receipt using OpenSSL

#if DEBUG
let result = PKCS7_verify(receipt, nil, store, nil, nil, PKCS7_NOCHAIN)
#else
let result = PKCS7_verify(receipt, nil, store, nil, nil, nil)
#endif
```

### StoreKit Test Framework

Introduces the ability to continuously test StoreKit automation and transactions with the following additions:

1. Enables you in code full control of the local StoreKit test environment. All the controls that you had manually are now exposed inside your tests.
2. Works alongside XCTest for extending unit and UI test coverage to your in-app purchases.
3. Additionally, you have the ability to [disable](https://developer.apple.com/documentation/storekittest/sktestsession/3579480-disabledialogs#) all the sheets and dialogs that would normally appear. Making your tests run to completion without waiting for user interaction. One example of this is the addition of [`clearTransactions()`](https://developer.apple.com/documentation/storekittest/sktestsession/3579476-cleartransactions#).
4. Ability to trigger renewal of a subscription right away. so your tests can validate if your app's subscription features continue to work across renewals.
5. Test the app's handling of both successful and failed purchases, interrupted and deferred purchases, transactions that initiate outside the running of your app as well as all sort of cases related to subscriptions.

When testing StoreKit flows it's important to [include](https://developer.apple.com/documentation/storekittest/sktestsession) the StoreKit configuration file so that's referenceable by SKTestSession.

After testing everything it's essential to sign into App Store connect and set up the in-app purchases so you can test everything using the sandbox environment.

## Enhancements to the Sandbox environment

* Sandbox environment uses the same information from App Store Connect as when the app it's available in the App Store.
* Sandbox environment also requires an Apple ID for your test device.
* In Sandbox app receipts are signed by the App Store.
* Sandbox supports server-side receipt validation.
* Sandbox supports App Store server notifications.

You can manage and test your subscriptions inside the Sandbox configuration from Settings.app of your device where you configure your Sandbox account. Also, you can reset eligibility for introductory offers, so now there's **no need to set up a new Sandbox ID each time you want to retest an introductory offer**.

### App Store server notifications in the Sandbox Environment

![][app_store_server_notifications]

> App Store server notifications in Sandbox Environment

This session introduces two new notification types available in the Sandbox environment: `DID_CHANGE_RENEWAL_STATUS` & `DID_RENEW`

[app_store_server_notifications]: app_store_server_notifications.png



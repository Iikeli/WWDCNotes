# Explore testing in-app purchases

Learn how you can test in-app purchases throughout development with StoreKit Testing in Xcode, App Store sandbox, and TestFlight. Explore how each tool functions and how you can combine them to build the right workflow for testing your apps and games. We’ll also share a sneak preview of how you can test Family Sharing for in-app purchases & subscriptions in the App Store sandbox.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10142", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(Cecile-Lebleu)
   }
}



Speaker: Hemant Sawle, Commerce Developer Advocate 

## StoreKit Testing in Xcode

StoreKit Testing in Xcode allows local testing. Added in WWDC20.

- StoreKit configuration file to create and manage IAPs.
- Perform local testing of in app purchases
- Use simulator or device to test.
- Leverage StoreKitTest framework to automate testing IAPs.
- Sync products from AppStore Connect to Xcode: so configuration file doesn’t need to be created manually.
- Supports advance subscription use cases, like offer code redemption, price increase sheet, and subscriptions entering and exiting billing retry.
- Flexible subscription renewal rate, from real-time to every 2 seconds.

**New in Xcode 15:**

- Static renewal rates, independent of subscription duration.
- Simulate StoreKit errors to build better error handling.
- If running multiple instances of your app, the transaction manager will display transactions for each app instance.
- Transactions manager also allows buying IAPs directly, to test external transactions.

Learn more in the session *“What’s new in StoreKit 2 and StoreKit Testing in Xcode”* from WWDC23

## App Store Sandbox

App Store Sandbox allows testing products set up in App Store Connect on client and server.

Prerequisites:

- Accept Paid Applications agreement
- Registered device with your developer account
- Create Sandbox Apple ID to make purchases in App Store Connect > Users & Access
- Enable developer mode on device, in Privacy Settings

Sandbox enables testing production-like scenarios like purchases, restores, and subscription offers. Distribute the app directly to the device from Xcode, or using a distribution method like Release Testing, Debugging, or Custom (to generate an IPA file).

### Billing Problem message simulation (new)

- Available in sandbox, it will be available for customers in production when they enter billing retry.
- Uses StoreKit 2 message API with reason `billingIssue`.
- Implement a message listener in views to defer or suppress the message
- Simulate `billingIssue` message in sandbox to test how your app handles the message presentation.
- To trigger this, the sandbox Apple ID needs to be subscribed to an auto-renewable subscription. In the device, go to App Store Settings > Account Settings and disable "Allow Purchases & Renewals". Then, go back to your app and the billing issue message will appear.

Learn more about implementing StoreKit 2 Message API in *"What's new with in-app purchase"* from WWDC22

### Billing Grace Period (new)

Grace period allows users to maintain access to paid features while payment is being collected, without interruption.

- Go to App Store Connect > App > Subscriptions > Billing Grace Period.
- The durations shown only apply to production; in testing, the sandbox account's renewal rate is used.

### Family Sharing (new)

Allow customers to share digital purposes with their family members.

- Go to App Store Connect > App > Subscriptions / Non-Consumable Products > enable Family Sharing
- Organize Sandbox Family Sharing in App Store Connect
- Make a purchase with sandbox Apple ID
- Go to App Store Connect > Users and access > Family Sharing
- To stop sharing, go to the device's settings, Account Settings > Family Sharing > Stop sharing

Family Sharing in Sandbox allows validating:

- Merchandise family-shareable products using StoreKit's `isFamilyShareable` 
- Validate app logic to entitle service for family members
- Revoke access for family members, validate with `revocationDate` available in JWSTransactions
- Receive App Store Server Notifications for family members

Learn more in the Tech Talk session *"Explore Family Sharing for in-app purchases."* 

### iOS sandbox Account Settings (new)

Options only available in App Store Connect are now available on-device for testing. Go to App Store settings > Sandbox Account > Manage, to find Renewal Rate, Test Interrupted Purchases, and Clear Purchase History.

## TestFlight

TestFlight allows end to end beta testing and feedback from testers. Overview:

- Distribute app across all platforms
- Add internal and external testers
- Automatic updates
- Builds valid for 90 days

Learn more in the Tech Talk session *"Get started with TestFlight"*

When testing in-app purchases:

- Builds are downloaded using TestFlight app
- Uses Apple ID signed into Media & Purchases
- In-app purchases are free
- Subscription renewal rates are accelerated, equivalent to Sandbox
- If the app has implemented StoreKit's `showManageSubscription`, you can test subscription cancellation or change subscription.

**New:**

- Manage TestFlight testers: filter tester data like status, sessions, and bulk selection.
- Internal Only distribution makes the build only available to internal testers, and it cannot be submitted to App Store.
Learn more in the sessions *"What's new in App Store Connect"* and *"Simplify distribution in Xcode and Xcode Cloud"* from WWDC23.

# What’s new in App Store Connect

Discover the latest updates to App Store Connect, the suite of tools used to manage and submit apps to the App Store. Explore how you can use the latest features to test, price, promote, and automate the management of your app more easily. We’ll also share enhancements to tools like TestFlight and the App Store Connect API.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10117", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



**Speaker:** Laurel McAndrews

**Fun fact:** In this session visionOS is referred to as xrOS.

## SubscriptionStoreView

**NEW:** A new `SubscriptionStoreView` in SwiftUI to show the available subscriptions and make subscribing easier. It is possible to customise the content of the view, like backgrounds, buttons, and styles to achieve a design that is seamless with your app.
*See more in the session "Meet StoreKit for SwiftUI".*

## Price changes

In the spring, Apple introduced (900) new price points and the possibility to set a base region from which to generate prices in other regions.

*See more in the session "What's new in App Store pricing".*

## Manage testers and builds

The list of testers in App Store Connect shows a status for the testers, how many sessions and crashes they have had and the amount of feedback they have provided.

**NEW:** A new column is added which will show how the most recent device and OS on which the beta app was installed.

**NEW:** Is is possible to filter by tester data in order to view and manage segments of testers.

**NEW:** Bulk selction of testers to resend invitations, add to a group or remove testers altogether.

All the data will be available through the App Store Connect API.

### Internal builds

**NEW:** When distributing builds from Xcode, they can now be marked as "Internal" that can't be distributed to external testers or submitted for the App Store.

**NEW:** Internal builds will be marked in App Store Connect

### What to Test

**NEW:** It will be possible, via Xcode Cloud, to update the "What to Test" information for a build.
* This can be done by adding a plain text file to a TestFlight folder located in the same folder as your Xcode project or workspace.
* It can also be done by using a custom build script to maybe pull a list of commit messages.

*See more in the session "Simplify distribution in Xcode and Xcode Cloud".*

## Family Sharing in Sandbox testing

**NEW:** You can create families of (up to 6) sandbox testers in App Store Connect.

**NEW:** "Sandbox on device", which allows you to view Family Group, modify renewal rate, test interrupted purchases and clear purchase history.

*See more in the session "Explore testing in-app purchases".*

## Building store presence

### App Store privacy nutrition labels

**NEW:** With the introduction of Vision Pro, some new types as been added to the app privacy questions. These can also be relevant on other platforms, fx if you use ARKit collect this data.
* Enable "Environment Scanning" if you collect data on a user's surroundings.
* Enable "Hands" if you collect hand structure or movements.
* Enable "Head" if you collect head movement.

*See more in the sessions "Explore App Store Connect for spatial computing" and "What's new in privacy".*

### Pre-order

**NEW:** You can now use pre orders on a regional basis, and launch in some regions and not others.

**NEW:** Redesigned "App Availability" page.

**NEW:** App Store Product page optimization test will now first stop when you choose to stop them.

*See more in the session "What's new in App Store pre-orders".*

## App Store Connect API

**NEW:** APIs are coming this year for setting up Game Center features.
* Create, configure and archive leaderboards and achievements.
* Submit scores and achievement unlock events via a server-to-server API.
* Remove scores and players from your leaderboards to automate management of fraudulent activity.
* Match players using custom rules like skill level or region, an upgrade to our matchmaking capabilities.

**NEW:** Generate marketing and customer service API keys.

**NEW:** Create user-based API keys.

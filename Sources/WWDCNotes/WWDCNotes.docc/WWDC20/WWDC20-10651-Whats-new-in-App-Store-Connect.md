# What's new in App Store Connect

Discover the latest improvements to App Store Connect, your suite of tools to upload, submit, and manage apps on the App Store. Learn about enhancements to the App Store Connect API, in-app purchase and subscriptions, Game Center, and more.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10651", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(thecodedself)
   }
}


## App Clips
* Can be beta tested on TestFlight
* For production, you'll need to set up App Clip Card Metadata and Associated Domains.
* App clips are packaged with your app itself when uploading to App Store Connect.

### Invocation
A Safari App Clip Banner shows on an Associated Domain. Tapping the banner's Open button opens an App Clip Card containing the App Clip's metadata. Tapping the Open button on the card opens the App Clip itself.

* Every invocation has an Invocation URL, which can act as a deep link into specific functionality (the URL components can act as parameters).
* For beta testing, you can provide three Invocation URLs to a build in the TestFlight section of App Store Connect 
* In the TestFlight app, you will see an App Clip section in the app detail view with each Invocation URL you configured for the build. You can test app clips directly from here.

## Game Center

* You can now allow **Challenges** which allows players to challenge each other to complete an achievement or to beat a leaderboard score.
* **Recurring Leaderboards** let you collect scores for a preset period of time and then repeat it at specific intervals. For example, every Friday night from 17:00 to 22:00.

## Subscriptions

* You can configure **Family Sharing** for auto-renewable subscriptions and non-consumable in-app purchases
* Once enabled for a live in-app purchase, you cannot turn it off again.

## App Store Connect API

Two new categories of API functionality have been added, with a total of more than 200 new endpoints.

App Metadata:

* Create a new version
* Set app pricing
* Edit app and version metadata
* Associate a build with the version
* Submit for App Review

Power and Performance:

* Metrics and diagnostics
* Download the same aggregate data that drives the new Power and Performance analysis tools in Xcode

# Explore App Store Connect for spatial computing

App Store Connect provides the tools you need to test, submit, and manage your visionOS apps on the App Store. Explore basics and best practices for deploying your first spatial computing app, adding support for visionOS to an existing app, and managing compatibility. We’ll also show you how TestFlight for visionOS can help you test your apps and collect valuable feedback as you iterate.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10012", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speakers: Justin Thomas, App Store Connect Engineer and Maciej Kujalowicz, TestFlight Engineer

## Set up your app

Three options to choose from when settuing up your app:

- Create new app (if it a new app or if it should have different pricing or availability then your other apps)
- Add to existing app (allows sharing of in-app purchases across platforms)
- Manage compatible iPad and iPhone apps (make current apps available for visionOS)

### Creating a new app

Create the visionOS app the same way that you normally add new apps in App Store Connect but select "visionOS" as platform.

### Add to existing app

Add a new platform in the sidebar on the app page in App Store Connect and select "visionOS" as platform.

### Manage compatible iPad and iPhone apps

> If there's any reason why you think your app doesn't make sense on the platform, you can manage its availability on visionOS.

Click the ellipsis button in top of the "Apps" page in App Store Connect and select "iOS Apps on visionOS Availability" from the menu.
In the new view you can select which apps are made available for visionOS.

The same can be done in the "Pricing and Availability" section of the app page for your app.

> If your compatible iPad and iPhone app is made available by using this option and you later add the visionOS platform, releasing it will replace the iOS app version on the App Store.

*See more in the sessions "Run your iPad and iPhone apps in the Shared Space" and "Enhance your iPad and iPhone apps for the Shared Space".*

## Beta test with TestFlight

> ...today we are introducing support for visionOS.

### New to TestFlight?

*Learn more about TestFlight from the Tech Talks "Get started with TestFlight".*

*See more in the session "Explore Xcode Cloud workflows" from WWDC21.*

### Distribute builds

Distributing builds for visionOS works the same way as distributing builds for iOS, macOS and tvOS.

> Each [beta tester] group has an option to enable or disable the ability to install iPhone and iPad apps by testers from this group on the headset.
>
> This option can help you expand the team of testers while you progress with testing the compatibility of your iOS apps.

### Install apps

- In the TestFlight app on visionOS, the list in the sidebar includes all visionOS and iOS apps that can be installed an tested on the device.
- Applications not compatible to install are listed in a separate category of iOS-only apps.
- A toggle on the top of the app page allows the tester to switch between the iOS app and visionOS app.
- The TestFlight app basically works the same way as on iOS and iPadOS.

### Collect feedback

- Press the Digital Crown and top buttons at the same time to capture screenshots.
- On the app's page in the TestFlight app there is a "Send Feedback" button where screenshots can be attached.
  - Screenshots can be cropped before sending.
- If beta app crashes an alert appears suggesting to send feedback and share more information.
- Feedback is available in App Store Connect and the Xcode Organizer.

#### Tester engagement metrics

> App Store Connect web and mobile also provide information about how many testers installed and launched a specific version of your app, or how many of them submitted crash or screenshot feedback.

## Get ready for the App Store

- New data types for Privacy Nutrition labels.

> - Check “Environment Scanning” if your app collects data about the user's surroundings, such as mesh, planes, scene classification, or image detection of the user's surrounding.
> - Check “Hands” if your app collects data about the user's hand structure and hand movements.
> - Check “Head” if your app collects data about the user's head movement.

## See more

> Check out the "What's new in App Store Connect" session to learn more about updates across App Store Connect.
>
> Also, see "Get started with building apps for spatial computing" for development tips.

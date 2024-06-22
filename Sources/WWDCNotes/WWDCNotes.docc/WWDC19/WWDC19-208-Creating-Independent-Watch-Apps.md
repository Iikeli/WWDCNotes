# Creating Independent Watch Apps

watchOS 6 enables a whole new level of watchOS experiences by allowing fully independent apps and apps built just for Apple Watch, and by bringing the App Store to Apple Watch. Discover how to leverage the power of many iOS frameworks and technologies, now on watchOS, to create fully independent experiences on Apple Watch.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/208", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



Watch apps can now be independent - the iPhone app is now optional.

Up to watchOS 5, the Watch app was embedded in the iOS app and both were downloaded to the iPhone. iPhone then handled the task of installing the Watch app to the Watch.

Now, in iOS 13 and watchOS 6, both apps are installed straight from the App Store directly on the relevant device, each device installs its own app. This applies to all apps in the store today. iOS app no longer has the Watch app bundled inside, it no longer counts towards your iOS app cellular download limit if you don't need it.

This also enables asset/variant thinning for Watch apps - if you have a Series 4 watch, only Series 4 assets are downloaded to the watch (previously the iOS app had to include all variants in the bundled Watch app for any possible watches).

All current apps belong to the category of “dependent apps”, i.e. the Watch app depends on the iOS app. If you install a dependent app on the Watch, the iPhone will automatically install the matching companion iOS app. (Watch app launch is blocked until the iOS app is installed.)

On the other hand, independent apps can live without their iOS counterpart installed. They are backwards-compatible with earlier OSes.

- to make an app independent, check the checkbox “Supports Running Without iOS App Installation”

Watch-only apps - apps that do not even have an iOS counterpart - are also possible, they require watchOS 6.


## Other improvements

Debugging in the simulator is now up to 10x faster, on the device up to 2x faster.

Controls:

- added text field control ([`WKInterfaceTextField`](https://developer.apple.com/documentation/watchkit/wkinterfacetextfield)) to let you implement sign in forms (use [`WKAlertAction`](https://developer.apple.com/documentation/watchkit/wkalertaction) for "terms & conditions")
- Sign In With Apple button ([`AuthenticationServices`](https://developer.apple.com/documentation/authenticationservices), [`WKInterfaceAuthorizationAppleIDButton`](https://developer.apple.com/documentation/watchkit/wkinterfaceauthorizationappleidbutton))

Added continuity keyboard for Watch ⭤ iOS - you can e.g. enter passwords to log in on the watch using your iOS device.

- for proper handling of logging in, set [`textContentType`](https://developer.apple.com/documentation/watchkit/wkinterfacetextfield/3120036-settextcontenttype) and associated domains

Getting health authorization is now also supported on watchOS.


### Push notifications

Watch is now a standalone push target, you can send user-visible notifications and background notifications straight to the watch.

New APNs request header - `apns-push-type`: set to `alert` for user-visible notifications, and `background` for background notifications.

- required for watchOS

Added notification service extension support for e.g. decrypting notifications.

Complication pushes can also be sent straight to the watch ([PushKit](https://developer.apple.com/documentation/pushkit)).


### Networking

It’s now preferred to use URLSession than WatchConnectivity.

- WC is still available for any iOS-to-watchOS specific communication, but only use it if you really need to
- check [`WCSession.isCompanionAppInstalled`](https://developer.apple.com/documentation/watchconnectivity/wcsession/3235766-iscompanionappinstalled)
- make sure to use background sessions

CloudKit - full [`CKSubscription`](https://developer.apple.com/documentation/cloudkit/cksubscription) / CloudKit notifications support.

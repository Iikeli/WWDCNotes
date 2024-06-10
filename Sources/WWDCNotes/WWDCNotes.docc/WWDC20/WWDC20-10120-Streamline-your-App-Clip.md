# Streamline your App Clip

App Clips are best when they provide an “in the moment” experience for people using them, like ordering your favorite refreshing beverage or paying for parking. We’ll share guidelines and best practices for building focused and consistent App Clips, show you how to streamline transaction experiences by taking advantage of technologies like App Clip notifications and location confirmation, and explore how you can help people move from your App Clip over to your full app. 

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10120", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Best practices

- interactions with clips need to be quick and focused
- focus on essential tasks
- when the App Clip launches, it should be usable right away (no splash screens/downloads before the user can start using the App Clip)
- ask the user to sign-in after they completed their task
- the main app should provide the same streamline experience of the App Clip (especially since the App Clip Experience will be launched in the app if the user has the app installed)
- use Sign in with Apple or [`ASWebAuthenticationSession`][ASWebAuthenticationSession] to authenticate the user
- App Clips can request permissions for camera, microphone, and Bluetooth, these grants will be transferred to the main app if the user downloads it

## Streamlining transactions

### Location

Instead of asking for location access, use specific URLs in nfc tags inside your business: this way you can skip this step as you know the user has scanned the tag at your business.

In order to make this more secure, beside the nfc tag, when your App Clip receives the payload from a physical nfc tag, you can ask the system if the payload has been acquired at a specific location: this is there's no user location access prompt  and it's displayed in a callout in your App Clip Card:

![][locationCardImage]

To opt-in into this behavior, set the boolean flag to 1 into your `info.plist` for the key [`NSAppClipRequestLocationConfirmation`][NSAppClipRequestLocationConfirmation].

After the App Clip launches, you can ask the system for confirmation via the following code:

```swift
import AppClip

guard let payload = userActivity.appClipActivationPayload else {
    return
}

let region = CLCircularRegion(
  center: CLLocationCoordinate2D(
    latitude: 37.3298193,        
    longitude: -122.0071671
  ), 
  radius: 100, 
  identifier: "apple_park"
)

payload.confirmAcquired(in: region) { inRegion, error in
  // inRegion == true means that the verification succeeded.
}
```

### Notifications

Similar to location, App Clips can bypass the prompt for notification authorization by setting the [`NSAppClipRequestEphemeralUserNotification`][NSAppClipRequestEphemeralUserNotification] boolean key to 1.

This will be shown to the user in the App Clip Card as well:

![][notifiCardImage]

To verify if the user has granted this authorization:

```swift
import UserNotifications

let center = UNUserNotificationCenter.current()

center.getNotificationSettings { (settings) in
  if settings.authorizationStatus == .ephemeral {
    // User has already granted ephemeral notification.
  }
}
```

Ephemeral (local) notifications can be sent up to 8 hours after the App Clip launch.

## Transition users to your app

After the user has downloaded the main app, you can transfer data from your App Clip to the main app.

The way to do so is via a secured App Group.

For example, this is how you use the App Group to store the Sign In With Apple credentials:

```swift
// App Clip code
// Automatically log in with Sign in with Apple
import AuthenticationServices

SignInWithAppleButton(.signUp, onRequest: { _ in
}, onCompletion: { result in
    switch result {
    case .success(let authorization):
        guard let secureAppGroupURL = 
            FileManager.default.containerURL(forSecurityApplicationGroupIdentifier:
                "group.com.example.apple-samplecode.fruta")
            else { return };
        guard let credential = authorization.credential as? ASAuthorizationAppleIDCredential 
            else { return }
        save(userID: credential.user, in: secureAppGroupURL)
    case .failure(let error):
        print(error)
   }
})
```

In the main app this is how you fetch those credentials:

```swift
// Main app code

import AuthenticationServices

let provider = ASAuthorizationAppleIDProvider()
guard let secureAppGroupURL =
    FileManager.default.containerURL(forSecurityApplicationGroupIdentifier:   
        "group.com.example.apple-samplecode.fruta")
    else { return };
let user = readUserID(in: secureAppGroupURL)
provider.getCredentialState(forUserID: user) { state, error in
    if state == .authorized {
       loadFavoriteSmoothies(userID: user)
   }
}
```

[ASWebAuthenticationSession]: https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession
[NSAppClipRequestLocationConfirmation]: https://developer.apple.com/documentation/bundleresources/information_property_list/nsappclip/nsappcliprequestlocationconfirmation
[NSAppClipRequestEphemeralUserNotification]: https://developer.apple.com/documentation/bundleresources/information_property_list/nsappclip/nsappcliprequestephemeralusernotification

[locationCardImage]: WWDC20-10120-locationCard
[notifiCardImage]: WWDC20-10120-notifiCard
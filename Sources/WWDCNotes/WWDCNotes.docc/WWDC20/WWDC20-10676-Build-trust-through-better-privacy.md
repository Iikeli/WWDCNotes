# Build trust through better privacy

Privacy is a more important issue than ever. Learn about Apple’s privacy pillars, our approach to privacy, and how to adopt the latest features on our platforms that can help you earn customer trust, create more personal experiences, and improve engagement. Explore the transparency iOS provides when your app is recording using the microphone or camera, control over location with approximate location, tracking transparency and permissions, and much more.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10676", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Privacy pillars

- On-device processing: no external server involved
- Data minimization: request only user data that you actually need
- Security protection: enforce the privacy protections on Apple's platform
- Transparency and Control: provide the user understanding and control over their data.

## On-device processing

Here are plenty of Apple Examples:

- All continuous Machine Learning models improvements are done on device:
  - QuickType Keyboard
  - QuickType Quik Reply
  - Hey Siri Vocal Classifier
  - Photo Sharing
  - DIctation Language Models
  - HomeKit Security Video Object Detection
- On-device dictation
- HomeKit face recognition (opt-in only)
- Fraud Prevention
- Sleep Mode
- Mobility Metrics
- Sound Detection
- Siri suggestions
- Translate App
- Approximate Location
- Tips
- Smart Widgets
- Spatial Audio
- Handwashing
- Audio Exposure
- Smart Automations

## Data minimization

- Do not ask a user to share lots of personal information for features that will only take advantage of a little.

- Trust is built over time, and starting out by showing respect for users data by asking for access to as little as possible is a great first step.

### Photos

New from this year is the Limited Photos Library:  
users can give apps access to only a limited selection of their photos instead of their entire photos library.

This is the new iOS 14 prompt when apps ask for photo library access:

![][photoAccessImage]

If possible use the new `PHPicker` instead of `UIImagePickerController`, this skips the photos library access entirely.

For more information, refer to the [`Handle the Limited Photos Library in Your App`][20-10641] session.

### Location

From iOS 14 users can choose to share only their _approximate_ location with an app.

This is the new prompt:

![][locationAccessImage]

Your app can ask for approximate location by default by setting the `NSLocationDefaultAccuracyReduced` `info.plist` key to `true`.

Apps can ask for a temporary upgrade to precise location as well:

![][tempImage]

For more information, refer to the [`What's New in Location`][20-10660] and [`Design for Location Privacy`][20-10162] sessions.

### Contact

From iOS 14 the system will suggest auto completion with contacts details as well.

The user just needs to start typing the name of a contact and the keyboard will suggest to complete all details automatically, no need for the app to have contact access.

![][contactsAutoFillImage]

To get this behavior we need to set the `UITextField` [`textContentType`][contentTypeDoc] property.

For more information, refer to the [`Autofill Everywhere`][20-10115] session.

## Security

### Server name tracking

Until iOS 13 DNS queries were made in plain sight and anyone in between could see where/what the user is visiting. 

From iOS 14 (and equivalent in other platforms) the system uses [Dot (DNS over TLS)][dotWiki] and [DoH (DNS over HTTPS)][dohWiki], which encrypt these queries, making sure that no 3rd parties can access to what the system is querying.

For more information, refer to the [`Enable Encrypted DNS`][20-10047] session.

## Transparency

### App Store transparency

While App are already required to have a Privacy Policy within the app itself, from fall 2020, apps will be required to expose such policies in the App Store as well.

![][dataUsageImage]

This is done via a a questionnaire to be filled in App Store Connect.

3rd party SDKs are considered part of your app, therefore you will need to declare what data they collect and how it is used.

### Intelligent Tracking Prevention (ITP) Enhancements

While Apple platforms have been using ITP since iOS 11, this year we have even more transparency with as we can see what known trackers ITP is protecting you from right from Safari's toolbar:

![][trackMeImage]

### App Pasteboard

From iOS 14 the user will see a pop up every time the pasteboard is accessed:

![][pasteboardImage]

### Recording Indicator

When the camera or the microphone is turned on, a new indicator will be displayed in the status bar.

![][indicatorStatusBarImage]

Control Center will additionally show which app is currently using the camera or microphone or which app has recently used it. 

### Local network access

Accessing the network lets an app see who else is in the same network, what devices are available etc:  
with this information an app can profile the user and understand if the user is at home and more details.

From iOS 14, accessing the local network (e.g. via Bonjour or mDNS scan) will trigger a prompt to the user requesting permission:

![][networkImage]

You should declare which Bojour services you need access to in the `info.plist` and the usual usage string.

For more information, refer to the [`Support Local Network Privacy in Your App`][20-10110] session.

## Private Wi-Fi address

Since iOS 8 the phone uses MAC randomization when it is not connected to Wi-fi. 

However when is is connected is uses the real MAC address, leaving trails of their connectivity.

With iOS 14 each wifi the device connect to will get a random MAC, which is generated daily as well. This is possible to turn off in the Wi-fi settings.

### Nearby Interaction framework

The [`NearbyInteraction`][niDoc] is a new framework that takes advantage of the U1 chip, 

To use this framework there's a prompt per session-based access.  
The data will be available while the app continues to be used in the foreground. 

For more information, refer to the [`Meet Nearby Interaction`][20-10668] session.

### App Clips

Have new location and notification permissions which are automatically granted without asking for permission via popup, instead, they're displayed in the App Clip card, before the user opens the App Clip.

For more information, refer to the [`Streamline your app clip`][20-10120] session.

### Safari Extensions

New in Safari 14, users will be able to select which websites a Safari Web extension gets access to and customize it to their needs.

![][safariExtImage]

For more information, refer to the [`Introducing Safari Web Extensions`][20-10120] session.

### Updates on MacOS

Many of the iOS access grants popup are brought over to macOS, for example:

- Bluetooth
- Limited Photos Library
- HomeKit
- Media and Apple Music
- CNCopyCurrentNetworkInfo

### Tracking transparency and control

The App Store policy require user permission for tracking across apps and websites ownder by other companies, this includes:

- Targeted advertising
- Advertising measurement
- Sharing with data brokers

If the app does any of this, it is oblidged to show the following popup:

![][trackTranspImage]

Exceptions:

- Linking is done solely on the user device
- Sharing with a data broker solely for fraud detection, prevention, or security

To show the popup, you need to use the ['AppTrackingTransparency'][appTrackDoc] framework. This also requires the `NSUserTrackingUsageDescription` `info.plist` key to be filled in.

In addition, users are able to choose to not be asked by any app to be tracked:

![][dontTrackImage]

### Campaign Tracking

[`SKAdNetwork`][skAdDoc] helps advertisers measure the success of ad campaigns while maintaining user privacy.

For more information, refer to the [`What's New with in App Purchases`][20-10661] session.

[20-10660]: ../10660
[20-10162]: ../10162
[20-10641]: ../10641
[20-10120]: ../10120
[20-10115]: https://developer.apple.com/videos/play/wwdc2020/10115
[20-10047]: ../10047
[20-10110]: https://developer.apple.com/videos/play/wwdc2020/10110
[20-10668]: https://developer.apple.com/videos/play/wwdc2020/10668
[20-10661]: https://developer.apple.com/videos/play/wwdc2020/10661
[contentTypeDoc]: https://developer.apple.com/documentation/uikit/uitextcontenttype
[dotWiki]: https://en.wikipedia.org/wiki/DNS_over_TLS 
[dohWiki]: https://en.wikipedia.org/wiki/DNS_over_HTTPS
[niDoc]: https://developer.apple.com/documentation/nearbyinteraction
[appTrackDoc]: https://developer.apple.com/documentation/apptrackingtransparency
[skAdDoc]: https://developer.apple.com/documentation/storekit/skadnetwork

[photoAccessImage]: photoAccess.png
[locationAccessImage]: locationAccess.png
[tempImage]: temp.png
[contactsAutoFillImage]: contactsAutoFill.png
[dataUsageImage]: dataUsage.png
[trackMeImage]: trackMe.png
[pasteboardImage]: pasteboard.png
[indicatorStatusBarImage]: indicatorStatusBar.png
[networkImage]: network.png
[safariExtImage]:  safariExt.png
[trackTranspImage]: trackTransp.png
[dontTrackImage]: dontTrack.png
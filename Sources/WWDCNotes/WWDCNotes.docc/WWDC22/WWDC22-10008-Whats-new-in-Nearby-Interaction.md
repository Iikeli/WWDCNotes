# What's new in Nearby Interaction

Discover how the Nearby Interaction framework can help you easily integrate Ultra Wideband (UWB) into your apps and hardware accessories. Learn how you can combine the visual-spatial power of ARKit with the radio sensitivity of the U1 chip to locate nearby stationary objects with precision. We’ll also show you how you can create background interactions using UWB accessories paired via Bluetooth.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10008", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample app](https://developer.apple.com/documentation/nearbyinteraction/finding_stationary_objects_with_precision)

## ARKit-enhanced Nearby Interaction 

- leverages the same underlying technology that powers Precision Finding with AirTag
- available via Nearby Interaction
- distance and direction information is more consistently available than using Nearby Interaction alone
- best for use cases that need to guide a user to a specific nearby object
- best used for interacting with stationary devices
- Requires `NSCameraUsageDescription` purpose string in <kbd>Info.plist</kbd>

Enable this via the new [`NINearbyPeerConfiguration`][NINearbyPeerConfiguration]'s [`isCameraAssistanceEnabled`][isCameraAssistanceEnabled] property

```swift
let session = NISession()

let configuration = NINearbyPeerConfiguration(..)
configuration.isCameraAssistanceEnabled = true // 👈🏻

session.run(configuration)
```

⚠️ Only a single `ARSession` can be running for a given application.

If you already have an ARKit experience in your app, it is necessary to share that `ARSession` with the `NISession`:

```swift
let niSession = NISession()

let arView = ARView(frame: .zero)
niSession.setARSesion(arView.session) // 👈🏻
```

## Background sessions

Until now, when the app transitions to the background, or when the user locks the screen on iOS and watchOS, any running `NISession`s are suspended until the application returns to the foreground.

In iOS 16, we can connect to devices via BLE and then run a NISession in the background:

```swift
var niSession = NISession()

func runBackgroundSession(accessoryData: Data, peripheral: CBPeripheral) {
  // Provide the accessory's UWB configuration data, and its Bluetooth peer identifier
  let peerIdentifier = peripheral.identifier
  let config = NINearbyAccessoryConfiguration(
    accessoryData: accessoryData,
    bluetoothPeerIdentifier: peerIdentifier
  )
  session.run(config)
}
```

This helps create "hands-free" experiences, like having music start playing as soon as the user walks into a room.

This background mode requires the `Nearby Interaction` string in the `UIBackgroundModes` array in your app's <kbd>Info.plist</kbd>

## Third-party hardware

- U1-compatible development kits out of beta
- Updated specification for accessory manufacturers

Check the [developer portal][ni] for more details.

[NINearbyPeerConfiguration]: https://developer.apple.com/documentation/nearbyinteraction/ninearbypeerconfiguration
[isCameraAssistanceEnabled]: https://developer.apple.com/documentation/nearbyinteraction/ninearbypeerconfiguration/4013050-iscameraassistanceenabled
[ni]: https://developer.apple.com/nearby-interaction
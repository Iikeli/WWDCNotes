# Explore Nearby Interaction with third-party accessories

Discover how your app can interact with Ultra Wideband (UWB) third-party accessories when running on a U1-equipped device. We’ll show you how to use the Nearby Interaction framework’s standards-based technology to implement precise and directionally-aware experiences with accessories. Learn about resources for getting started with accessory and app development such as development kits, sample code, and specification documents, along with supported technology providers.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10165", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



The [Nearby Interaction framework][nif]:

- makes it easy to take advantage of the unique capabilities of U1, Apple's chip for Ultra-Wideband technology
- enables creating precise and spatially-aware interactions between nearby devices

iOS examples:

- Find My's <kbd>Precision Finding</kbd> with AirTag
- Handoff gestures between iPhone and HomePod mini

## Nearby Interaction quick start

1. Create a [`NISession` ][NISession] instance - this is the main object through which you'll configure and run your spatial interactions with nearby devices
2. Set the delegate on the `NISession` instance, which needs to be an object conforming to [`NISessionDelegate`][NISessionDelegate] - The delegate will receive updates from the framework
3. Create a configuration object (a [`NIConfiguration`][NIConfiguration] subclass) - E.g., [`NINearbyPeerConfiguration`][NINearbyPeerConfiguration]
4. Call run on the `NISession` instance with your configuration
5. The framework will start providing updates to your delegate - most importantly, you will get a stream of [`NINearbyObject`][NINearbyObject] updates, each containing `distance` and, optionally, `direction`, to nearby devices that are actively participating in the session

## Permission prompt updates

Starting from iOS 15, the permission prompt is similar to other system permissions like user location:

- the permission prompt will be shown on the first time your app runs an `NISession`
- the permission will only be shown only once per app install
- the permission can be changed in the Settings.app's Privacy page under <kbd>Nearby Interactions</kbd>

Add a `NSNearbyInteractionUsageDescription` entry in your app <kbd>Info.plist</kbd>.

## Third-party accessory API

- U1-compatible development kits
- Sample app code
- [Specifications for accessory makers are available here][spec]

### Configuring for accessory interaction

- Nearby Interaction expects your app and your accessory to have some sort of capability to exchange data between them - what technology to use to exchange data is entirely up to you (e.g., Bluetooth, LAN, Internet)
- To start a session with an accessory, you'll create a [`NINearbyAccessoryConfiguration`][NINearbyAccessoryConfiguration]
- This configuration expects an <kbd>Accessory Configuration Data</kbd>, which the accessory needs to send to the iDevice via the data channel (before connecting via the Nearby Interaction framework), this data will be known by the accessory manufacturers.

```swift
/// Handle configuration data from the accessory.
private func setupAccessory(_ configData: Data, name: String) {
  do {
  config = try NINearbyAccessoryConfiguration(data: configData)
  } catch let error {
    print("Bad config data from accessory \(name). Error: \(error)")
    return
  }
  // Cache the token to correlate updates with this accessory.
  cacheToken(config! .accessoryDiscoveryToken, accessoryName: name)
}
```

Once we have our `NINearbyAccessoryConfiguration` ready, we can start a `NISession` and start the session.

- Similarly to how Nearby Interaction needed configuration data from the accessory, the accessory also needs configuration data from Nearby Interaction in order to know how to configure itself. This data needs to be in a format called <kbd>Shareable Configuration Data</kbd>
- When you run a session with an accessory configuration, Nearby Interaction will provide the Shareable Configuration Data to your app through a delegate callback
- You'll use your data channel to send the shareable configuration data back to the accessory

```swift
/// Send the Shareable Configuration Data to the accessory.
func session(_ session: NISession, didGenerateShareableConfigurationData: Data, for object: NINearby0bject) {
  // Get the data link for this accessory from a helper function.
  guard let conn = getConnection(object: object) else { return }
  // Send shareable configuration data.
  conn.sendShareableConfigurationData(data)
}
```

Once the code running on the accessory receives the data, it needs to provide it as-is, and as quickly as possible, to the Ultra-Wideband hardware on board.

Both the <kbd>Sharable COnfiguration Data</kbd> and <kbd>Accessory Configuration Data</kbd> are part of [Apple's specification][spec].  
The document is intended for chipset and module manufactures, and it contains the necessary details for creating Ultra-Wideband solutions that use industry standards to interoperate with U1 in iPhone.

[spec]: https://developer.apple.com/nearby-interaction/
[NISession]: https://developer.apple.com/documentation/nearbyinteraction/nisession
[NISessionDelegate]: https://developer.apple.com/documentation/nearbyinteraction/nisessiondelegate
[NIConfiguration]: https://developer.apple.com/documentation/nearbyinteraction/niconfiguration
[NINearbyPeerConfiguration]: https://developer.apple.com/documentation/nearbyinteraction/ninearbypeerconfiguration
[NINearbyObject]: https://developer.apple.com/documentation/nearbyinteraction/ninearbyobject
[NINearbyAccessoryConfiguration]: https://developer.apple.com/documentation/nearbyinteraction/ninearbyaccessoryconfiguration
[nif]: https://developer.apple.com/documentation/nearbyinteraction
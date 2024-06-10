# Meet Nearby Interaction

The Nearby Interaction framework streams distance and direction between opted-in Apple devices containing the U1 chip. Discover how this powerful combination of hardware and software allow you to create intuitive spatial interactions based on the relative position of two or more devices. We'll walk you through this session-based API and show you how to deliver entirely new interactive experiences — all with privacy in mind.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10668", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

iPhone's 11 (and later) U1 chip give iPhones special awareness.

Apple uses it for airdrop:  
point your device to the user you want to share to and that user will automatically be highlighted on top of the screen:

![][airdropImage]

### NearbyInteractions

[`NearbyInteractions`][niDoc] is a new framework that behaves as an interface to spatial awareness in iOS.

### User Control & Transparency

To use this framework, the user must allow your app to use `NearbyInteractions`, if granted, your app will be able to use `NearbyInteractions` until it is quit.

To use this framework you will need at least two devices, therefore this permission must be granted by both users.

Once granted, the devices can start to understand how far apart they are and in which relative direction:

![][twoDevicesImage]

## Special Awareness in iOS

Nearby Interaction provides your app two main types of inputs:

- a measurement of distance between devices
- a measurement of relative direction from one to the other. 

When your app is running a Nearby Interaction session, it is able to get a continuous stream of updates containing distance and direction information.

These updates are bidirectional: both sides/devices of the session are learning about each other's relative position simultaneously.

Each device can run several sessions (aka bidirectional updates with different devices) at the same time: each session with one other peer.

### Discovery Tokens and Peer Discovery

- When two devices would like to start a Nearby Interaction session, they need to know how to discover each other, this is called **Peer Discovery**.
- Peer Discovery is accomplished in a privacy-preserving manner via a **Discovery Token**, [`NIDiscoveryToken`][tokenDoc].
- `NIDiscoveryToken` is a randomly-generated identifier for a given device in a particular Nearby Interaction session. Its lifetime is equal to the session lifetime.

The devices that want to start a session need to know each other discovery token, a way to exchange this token is via the `MultipeerActivity.framework`:

```swift
/// Per-session Discovery Token 

@available (iOS 14.0, *)
open class NIDiscoveryToken: NS0bject, NSCopying, NSSecureCoding {
}

// Encode discovery token
if let encodedData = try? NSKeyedArchiver.archivedData(
  withRootObject: myDiscoveryToken,
  requiringSecureCoding: true
) { 
  // Share encoded token using your app's networking layer.
  // EXAMPLE: using MultipeerConnectivity.framework
  mpcSession.send(encodedData, toPeers: [myPeer], with: .reliable)
}
```

## Getting Started

> For more, read the [official documentation][getStartedDoc].

- All Nearby Interactions are encapsulated in [`NISession`][nisessionDoc]s.
- You provide your session a [`NINearbyPeerConfiguration`][niconfDoc] object you would like it to run with.
- We get our own `NIDiscoveryToken` via our `NISession` object [`discoveryObject`][discObjPropDoc] property

This is an example of a typical class that manages the Nearby Interaction session:

```swift
// A session instance. Store in whichever data structure makes the most sense for your app.
var niSession: NISession?

// Instantiate a new session object and set the session's delegate.
func prepareMySession() {
  // Verify hardware support.
  guard NISession.isSupported else {
    print("Nearby Interaction is not available on this device.")
    return
  }
  
  // Create a new session for each peer, this creates the device's discovery token as well.
  niSession = NISession()

  // Set the session’s delegate.
  niSession?.delegate = self // This class of 'self' needs to conform to NISessionDelegate.
}

// Share the encoded discovery token to the peer you intend to interact with.
func sendDiscoveryTokenToMyPeer(myPeer: Any /* change to whichever type represents peers in your app */) {                                
	guard let myToken = niSession?.discoveryToken else {
		// The session object is not initialized or has been invalidated.
		return
	}

	if let encodedToken = try? NSKeyedArchiver.archivedData(withRootObject: myToken, requiringSecureCoding: true) {
		<# share token using your app's networking layer #>
	}
}

// Once you receive a token from the peer, create a configuration and run the session.
// This functions shows how to decode token data that was previously encoded using NSKeyedArchiver.
func runMySession(peerTokenData: Data) {
  guard let peerDiscoveryToken = try? NSKeyedUnarchiver.unarchivedObject(ofClass: NIDiscoveryToken.self, from: peerTokenData) else {
    print("Unexpectedly failed to decode discovery token.")
    return
  }

  // Create a session configuration using the discovery token received from the peer.
  let config = NINearbyPeerConfiguration(peerToken: peerDiscoveryToken)

  // Run the session with the configuration.
  niSession?.run(config)
}
```

## NISessionDelegate

The `NISession` session [`delegate`][delDoc] receives all updates regarding the session status:

```swift
public protocol NISessionDelegate : NSObjectProtocol {
  
  // Monitoring Peers
  optional func session(_ session: NISession, didUpdate nearbyObjects: [NINearbyObject])
  optional func session(_ session: NISession, didRemove nearbyObjects: [NINearbyObject], reason: NINearbyObject.RemovalReason)
  
  // Managing Interruption
  optional func sessionWasSuspended(_ session: NISession)
  optional func sessionSuspensionEnded(_ session: NISession)
  optional func session(_ session: NISession, didInvalidateWith error: Error)
}
```

- [`session(_:didUpdate:)`](https://developer.apple.com/documentation/nearbyinteraction/nisessiondelegate/3601171-session) receives updates about nearby devices.
- [`session(_:didRemove:reason:)`](https://developer.apple.com/documentation/nearbyinteraction/nisessiondelegate/3601170-session) will update you when the session is no longer interacting with a nearby object, it also comes with a [reason][reasonDoc] (currently either `.timeout` or `.peerEnded`). ⚠️ This notification is delivered on a best effort basis and may not always be received.
- the last three methods are around managing the session state: 
  - [`sessionWasSuspended(_:)`](https://developer.apple.com/documentation/nearbyinteraction/nisessiondelegate/3601173-sessionwassuspended) is called for example when the app goes in the backdround
  - [`sessionSuspensionEnded(_:)`](https://developer.apple.com/documentation/nearbyinteraction/nisessiondelegate/3601172-sessionsuspensionended) lets us know that we can resume the session (you must wait for this function to be called before trying so). Note that this lets us know that we can resume, it's up to us to decide if we want to do so.
  - [`session(_:, didInvalidateWith:)`](https://developer.apple.com/documentation/nearbyinteraction/nisessiondelegate/3571263-session) is called when the session has been invalidated. Once a session is invalidated we need to restart a new session from scratch (as the current Discovery Token has been invalidated as well)

### NINearbyObject

[`NINearbyObject`][objDoc] contains the update from a session, it provides three properties:

- `discoveryToken` to identify the session
- `distance` in meters (`Float`), indicating how far apart the devices are
- `direction` as a relative `simd_float3` vector, pointing at the other device from our device poit of view.

`distance` and `direction` are nullable: this happens when the confidence of those values are very low and the devices are out of the U1 chip field of view

## Best Practices

- Always verify device support, you can do so via [`NISession.isSupported`](https://developer.apple.com/documentation/nearbyinteraction/nisession/3601169-issupported).
- Get familiar with the U1 chip field of view (approximately the same as the Ultra Wide camera's field of view on the iPhone 11.)
![][fovImage]

- For optimal performance, devices should be held in the portrait orientation.
- If there is any kind of obstacles between the two devices, this will result into limited measurement availability.
- Test your app in the simulator

[airdropImage]: WWDC20-10668-airdrop
[twoDevicesImage]: WWDC20-10668-twoDevices
[fovImage]: WWDC20-10668-fov

[niDoc]: https://developer.apple.com/documentation/nearbyinteraction
[getStartedDoc]: https://developer.apple.com/documentation/nearbyinteraction/initiating_and_maintaining_a_session
[nisessionDoc]: https://developer.apple.com/documentation/nearbyinteraction/nisession
[niconfDoc]: https://developer.apple.com/documentation/nearbyinteraction/ninearbypeerconfiguration
[tokenDoc]: https://developer.apple.com/documentation/nearbyinteraction/nidiscoverytoken
[discObjPropDoc]: https://developer.apple.com/documentation/nearbyinteraction/nisession/3564775-discoverytoken
[delDoc]: https://developer.apple.com/documentation/nearbyinteraction/nisession/3564773-delegate
[reasonDoc]: https://developer.apple.com/documentation/nearbyinteraction/ninearbyobject/removalreason
[objDoc]: https://developer.apple.com/documentation/nearbyinteraction/ninearbyobject

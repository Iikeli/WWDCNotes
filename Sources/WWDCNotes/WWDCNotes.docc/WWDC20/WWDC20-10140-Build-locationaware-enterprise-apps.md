# Build location-aware enterprise apps

Develop location-aware enterprise apps for your business and personalize your employee’s everyday experience. Learn how Apple built the Caffe Macs app for its on-campus cafeterias using iBeacons and Location Services and how you can apply these tools and frameworks to your own apps, while preserving employee privacy. From there, discover how you can use localization to deliver a great experience for your international employees.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10140", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



This session is mostly an introduction to how to use Core Location (see related session below) plus some tips.

## Determine device support

Before asking for authorization, check whether the current device supports the feature you're interested in:

```swift
if CLLocationManager.isMonitoringAvailable(for: CLBeaconRegion.self) {
    // Supports region monitoring to detect beacon regions
}

if CLLocationManager.isRangingAvailable() {
    // Supports obtaining the relative distance to a nearby iBeacon device
}
```

If the device doesn't support what you're interested in, offer the user a list of options (e.g. a list of locations/cafes/..)

## iBeacon

- A beacon is a device that emits a signal which can be detected by the system and passed to your app
- Those signals can identify when a user is within a certain location
- Once we detect a specific beacon signal, we can trigger something in our app.

- When deploying your beacon hardware, you must program each beacon with an appropriate proximity UUID, major value and minor value.
- The values identify each of your beacons uniquely and make it possible for your app to differentiate between them.

- Use region monitoring to detect the presence of a beacon
- Use ranging to determine the proximity of a detected beacon

```swift
// Stage 1: Region Monitoring

func monitorBeacons() {
    if CLLocationManager.isMonitoringAvailable(for: CLBeaconRegion.self) {

        let constraint = CLBeaconIdentityConstraint(uuid: proximityUUID)

		// Contains the proximity UUID, major value and minor value, of the beacons that you want to detect.
        let beaconRegion = CLBeaconRegion(
            beaconIdentityConstraint: constraint,
            identifier: beaconID
        )
        
        self.locationManager.startMonitoring(for: beaconRegion)
    }
}
```

```swift
// Stage 2: Beacon Ranging

func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
    guard let region = region as? CLBeaconRegion,
        CLLocationManager.isRangingAvailable()
        else { return }
    
    let constraint = CLBeaconIdentityConstraint(uuid: region.uuid)
    manager.startRangingBeacons(satisfying: constraint)
    beaconsToRange.append(region)
}

func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
    ...
}

func locationManager(
    _ manager: CLLocationManager,
    didRangeBeacons beacons: [CLBeacon],
    in region: CLBeaconRegion) {
    
    guard let nearestBeacon = beacons.first else { return }
    let major = CLBeaconMajorValue(truncating: nearestBeacon.major)
    let minor = CLBeaconMinorValue(truncating: nearestBeacon.major)
    
    switch nearestBeacon.proximity {
    case .near, .immediate:
        displayInformation(for: major, and: minor)
        
    default:
        handleUnknownOrFarBeacon(for: major, and: minor)
    }
}
```
# Connect Bluetooth devices to Apple Watch

Discover how you can integrate data from Bluetooth accessories into Apple Watch apps and complications. Bluetooth devices can provide medical data, sports stats, and more to Apple Watch, and help people get more out of your software in the process. We’ll show you how to connect to these devices during Background App Refresh to display the most up-to-date information in your Apple Watch complications, provide an overview of Core Bluetooth on watchOS, and explore best practices for Bluetooth accessory design.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10005", purpose: link, label: "Watch Video (10 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What's new in watchOS 8

With watchOS 7:

- you can only connect to Bluetooth devices while your app is in the foreground or during a background session
- if you wanted to have a complication with information from your accessory, the complication could not be updated unless the user opened your app

With watchOS 8:

- you can connect your Bluetooth accessory during Background App Refresh, this enables you to: 
  - update your complications with information from your accessory
  - send local notification if the accessory battery is running low

To enable Bluetooth Background Refresh in your app, add <kbd>bluetooth-central</kbd> to <kbd>UIBackgroundModes</kbd> in your <kbd>Info.plist</kbd>

## Core Bluetooth differences between platforms

|   | iOS | macOS | tvOS | watchOS |
| --- | --- | --- | --- | --- |
| Background execution | ✅ | ✅ | ❌ | watchOS 8+ |
| Foreground execution | ✅ | ✅ | ✅ | ✅ | 
| Minimum connection interval | 15ms | 15ms | 30ms | 30ms |
| Role | Both | Both | Central | Central |
| Connections | HW | HW | 2 | 2 |
| State preservation and restoration | ✅ | ❌ | ❌ | ❌ |

## How your accessory communicates with Apple Watch

When the app is in the foreground: 

- the app tries to connect to your accessory as soon as an advertisement (from your accessory) is seen
- once the advertisement is received, a Bluetooth LE connection is established

If the app is not in the foreground, the only way for your app to connect to the accessory is via Background App Refresh:

- during a refresh your app initiate a connection to a known peripheral, and the next time an advertisement is received, Apple Watch will connect to your accessory
- as soon as you're finished retrieving data, you can terminate the Bluetooth connection and process the data.

Background App Refresh can happen at any time:

- your accessory should advertise as often as possible when it has an important update to display on Apple Watch
- one possible strategy is to buffer sensor data on your accessory. When it's close to the buffer limit, increase its advertisement rates to increase the chances of reconnection to Apple Watch
- as a guide, you should advertise at least every two seconds in ideal RF conditions
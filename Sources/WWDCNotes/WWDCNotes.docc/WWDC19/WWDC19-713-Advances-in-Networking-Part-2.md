# Advances in Networking, Part 2

Take your networking apps to the next level with advances in Bonjour, custom message framing handlers, and the latest in security. You’ll also learn how to understand your networking performance by collecting metrics, and how best to use the modern networking frameworks on Apple platforms.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/713", purpose: link, label: "Watch Video (61 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Bonjour
Bonjour is how you advertise and discover services on the network.

- Used anytime you print with AirPrint, connect to an airplane-enabled device, use HomeKit to automate your home, really anytime you are connecting to something without typing in an IP address or a host name. 
- Available on Linux, Android, Chrome OS, it's how Chromecast does discovery. Microsoft added Bonjour support to Windows 10 back in 2015.
- A few improvements behind the scenes.

## Building Framing Protocols

TL;DR: we can now create our own communication protocol that lets different devices communicate with each other efficiently. We define the payload etc.

Used to encapsulate and encode application messages.

Two steps:

1. Implement a reusable framing protocol (NWProtocolFramerImplementation). Implement a reusable piece of code that defines your message framing. This is the protocol. 
2.Add the framing protocol to a connection
Add that protocol into your connection's protocol stacks so that you can use it for connection establishment as well as sending and receiving message. 

## Collecting Metrics

We have now more info on our connections. 

## Status Update

For privacy, `CNCopyCurrentNetworkInfo` is more restricted now.

To access to this info, the app must also meet at least one of the criteria below:

- Apps with permission to access location
- Currently enabled VPN app
- `NEHotspotConfiguration` (only Wi-Fi networks that the app configured)

Otherwise, it returns nil.

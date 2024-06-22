# Advances in Networking, Part 1

Keep up with new and evolving networking protocols and standards by leveraging the modern networking frameworks on all Apple platforms and following best practices for efficiency and performance. In this session, learn about Low Data Mode, Combine in URLSession, WebSocket, and improvements to network mobility.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/712", purpose: link, label: "Watch Video (56 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Low data mode

New User preference to minimize data usage:

- Explicit signal to reduce network data use 
- Per Wi-Fi and Cellular network

System policy:

- Discretionary tasks deferred 
- Background App Refresh disabled

Application adoption suggestions:

- Reduce Image quality
- Reduce pre-fetching
- Sync less often
- Mark Background tasks as discretionary (will run when iOS says so)
- Disable auto-play
- Do not block user-initiated work

URLSession 

- Try large prefetch with `allowsConstrainedNetworkAccess = false`, which allow the `URLSession` to run even on low data mode.

- On failure with `error.networkUnavailableReason == .constrained` try Low Data Mode alternative If you get a failure and the error has a network unavailable reason of constrained, that indicates the operation failed because you're in Low Data Mode and the right thing to do there is to turn around and perform your Low Data Mode operation.

Network.framework 

- Set prohibitConstrainedPaths on NWParameters
- Check isConstrained on NWPath
- Handle path updates  

## Combine in URLSession

What is Combine?

- Combine processes values over time. 
- It consists of publishers, operators, and subscribers. 
- The chain is driven by the request sent from the subscriber. 
- In response to the request, publisher sends value down the chain. 

`DataTaskPublisher`

`URLSession publisher`, similar to `URLSession.dataTask(with:completionHandler:)`:

- lets us write code concise, linear, and less error-prone.
less duplication
- Cells can be subscribers, and on reuse we can cancel the previous tasks.
- Cool demo at 20-25min in.

## WebSocket

New `URLSessionWebSocketTask`, a new API in the Foundation framework. Works with existing URLSession:

```swift
// Create with URL 
let task = URLSession.shared.webSocketTask(with: URL(string: "wss://websocket. example")!)
task.resume() 

// Send a message 
task.send(.string("Hello")) { error in /* Handle error */ }

// Receive a message 
task.receive { result in /* Handle result */ } 
```

## Mobility Improvements

Wifi and mobile data switch got smarter once again.

- Wi-Fi Assist was applied to hight-level APIs like Network.Framework and URLSession in iOS 13.  
Whenever the wifi connection is bad or loss. it will switch to connect cellular.
- Multipath Transports
- `allowsExpensiveNetworkAccess = false`
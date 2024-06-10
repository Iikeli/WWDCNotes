# Optimize for 5G networks

5G enables new opportunities for your app or game through better performance for data transfer, higher bandwidth, lower latency, and much more. Discover how you can take advantage of the latest networking technology and Apple hardware to create adaptive experiences for your content that best suit someone’s data connection and optimize network traffic.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10103", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Each generation leap results in increased opportunities for app developers to push the envelope and deliver previously unheard-of experiences in your app

## Path to [5G](https://en.wikipedia.org/wiki/5G)

5G provides other advances beyond just more data at a faster rate:

- faster round-trip times between your app and your server - meaning faster and more synchronized transactions
- 5G’s ability to handle more devices concurrently - you can continue to deliver data-related services in your app, even in congested public environments
- ...and more

## Built-in intelligence

Today’s modern 5G cellular networks come primarily in two varieties, Non-Standalone (NSA) and Standalone (SA).

- Non-Standalone
  - built on the existing LTE core
  - can use both LTE and 5G links to schedule traffic
  - operates at frequencies below seven gigahertz
  - includes support for millimeter wave

- Standalone
  - built entirely on the new 5G core
  - operates at frequencies below seven gigahertz
  - includes support for millimeter wave
  - delivers improved latency performance over LTE

Thanks to <kbd>Automatic Switch to 5G</kbd> and <kbd>Smart Data Mode</kbd>, iOS takes performance, security, and power into consideration when choosing which connectivity to use - your app can focus on the user experience.

## Practical guidance for developers

- ignore network type
- use high-level (Apple's) frameworks to take full advantage of 5G
  - AVFoundation for Media
  - CallKit for VOIP
  - Network framework/Foundation's URLSession

- tune your application for constrained and expensive paths
  - in most cases, these are automatically derived by the system based on the active cellular plan
  - the user can also affect these by altering the <kbd>Data Mode</kbd> settings on their device - there are three options:
    - Allow More Data on 5G - indicating an inexpensive path, similar to Wi-Fi
    - Standard mode - which is usually considered as expensive
    - Low Data Mode - which is considered constrained

The concept of **expensive** and **constrained** are surfaced as properties in all our high-level networking frameworks - the value of these properties should be your only consideration when determining the type of network services available for your app.

- `URLSession`'s [`allowsExpensiveNetworkAccess`](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/3235752-allowsexpensivenetworkaccess)

```swift
private func fetchAsset(at fetchURL: URL, allowsExpensive: Bool) {
  var fetch = URLRequest(url: fetchURL,
                         cachePolicy: .reloadIgnoringLocalCacheData,
                         timeoutInterval: 60)
  fetch.allowsExpensiveNetworkAccess = allowsExpensive // 👈🏻
 
  // Fallback to low resolution fetch if network is expensive and unavailable
  URLSession.shared.dataTask(with: fetch) { [weak self] data, response, error in
    if let strongSelf = self,
      let error = error as? URLError,
      error.networkUnavailableReason == .expensive { // 👈🏻
        strongSelf.fetchAsset(at: strongSelf.lowResURLString, allowsExpensive: true)
    }
  }.resume()
}
```

> if your application implements policy based on expensive, provide a way for users to influence that policy

- In <kbd>Network.framework</kbd>, you have another option to check for constrained, expensive, and inexpensive path:

```swift
func onConnectionStateChange(_ connection: NWConnection) {
  guard let path = connection.currentPath else { return } // insure we have a path

  if connection.state == .ready {
    // Update your data usage heuristics based on cost for this connection
    
    if path.isConstrained {
      // Reduced Data Usage
    } else if path.isExpensive {
      // Default Data Usage
    } else {
      // Unrestricted Data Usage
    }
  }
}
```

- AVFoundation:
  - [`AVURLAssetAllowsConstrainedNetworkAccessKey`](https://developer.apple.com/documentation/avfoundation/avurlassetallowsconstrainednetworkaccesskey) - Allows network requests to use the constrained interface
  - [`AVURLAssetAllowsExpensiveNetworkAccessKey`](https://developer.apple.com/documentation/avfoundation/avurlassetallowsexpensivenetworkaccesskey) - Allows network requests to use the expensive interface

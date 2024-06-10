# Get ready for iCloud Private Relay

iCloud Private Relay is an iCloud+ service that prevents networks and servers from monitoring a person's activity across the internet. Discover how your app can participate in this transition to a more secure and private internet: We'll show you how to prepare your apps, servers, and networks to work with iCloud Private Relay.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10096", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What is iCloud Private Relay?

- new service that prevents networks and servers from monitoring user activity across the internet
- built into iOS and macOS
- available as part of every iCloud+ subscription (can be disabled by user or in certain regions)
- affects:
  - all Safari browsing
  - all DNS queries
  - all http (_not_ https) apps traffic 
    - if you app provides a content filter or a parental controls filter, it will still see traffic before it goes through Private Relay

Not applicable traffic:

- Local network connections
- Private domains
- Network Extensions and VPNs
- Proxy configurations

### Without iCloud Private Relay

- when a user accesses the internet, anyone on their local network can see the names of all of the websites they access based on inspecting DNS queries
  - this information can be used to fingerprint a user and build a history of their activity over time

- when connections reach the servers that run websites, those servers can see the user's IP address
  - this allows the servers to determine user location without explicit permission
  - the servers are able to fingerprint user identity and recognize users across different websites, even when tools are preventing correlation via cookies (e.g., Safari's Intelligent Tracking Prevention)

### With iCloud Private Relay

- iCloud Private Relay adds multiple secure proxies to help route user traffic and keep it private
- the proxies are run by separate entities
- one is Apple, and one is a content provider
- when a user accesses the internet, only the client IP address is visible to both the network provider and to the first proxy
  - the second proxy only sees the name the user is requesting and uses that to build the connection to the server
  - no one in this chain, _not even Apple_, can see both the client IP address and what the user is accessing

## (Best practices) Prepare your app

Most apps don't need to do anything.

- avoid insecure connections - Private Relay will add encryption up to the egress proxy 
- use `URLSession` and `NWConnection`
- you can use the <kbd>Network Xcode Instrument</kbd> to inspect your tasks, even when they are going through Private Relay
- you can use metrics APIs in both `URLSession` and <kbd>Network.framework</kbd> to understand when your connections use Private Relay:
  - `URLSessionTaskTransactionMetrics` - you can check if your task used a proxy
  - `NWConnection.EstablishmentReport` - you can inspect the timings of DNS name resolution and each stage of the proxied connection establishment

- prefer CoreLocation to IP address geolocation

## Prepare your server

- make sure your server supports TLS
- Proxy UP address pools
  - your servers can identify connections that come in using Private Relay by recognizing the proxy IP addresses
  - these addresses may be shared by many users within a region
  - each address is mapped to a specific city or region
    - if you apply the correct geo IP mapping databases, your servers will still have the relevant information

- stop solely relying on client IP address to determine user location or identity
- if you need location access, consider requesting the user's location explicitly (via <kbd>CoreLocation</kbd>)
- if you need to identify users, request a login or some other form of explicit identification rather than assuming that the IP address is tied to the identity

## Manage your network

- if you're running a packet trace on your local network when Private Relay is in use, you'll see some new traffic patterns:
  - you'll now see a lot more traffic running on UDP port 443
    - This is `QUIC` -- or `HTTP/3` -- traffic that's being used to communicate with the Private Relay proxy
    - you can make sure your traffic works well by allowing `UDP` port `443` on your network and by making sure your routers or Network Address Translators are tuned to handle it well

- 
  - you'll also see fewer cleartext UDP DNS queries on your network
    - with Private Relay, the device first sets up a connection to the ingress Proxy using `QUIC`, or `HTTP/3`
    - once the network connection is established to the ingress proxy, access to a server is secured within the connection to the ingress proxy
    - on the server end, there is no change in protocol

- if you need to audit/monitor all user traffic (e.g., if you run an enterprise or school network, your network may have policies that require intercepting all traffic), you can block the hostname of the iCloud Private Relay proxy server
  - when a device connects to your network, the user will receive a prompt indicating that Private Relay is blocked on the current network
    - they can then choose to either disable Private Relay for that network or switch networks

> For parental controls, the best solution is to use content filter APIs provided by NetworkExtension framework. This allows traffic to be audited on device even when Private Relay is enabled.
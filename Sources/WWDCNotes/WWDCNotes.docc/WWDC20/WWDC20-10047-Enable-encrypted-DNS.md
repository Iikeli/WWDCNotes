# Enable encrypted DNS

When people access the web within your app, their privacy is paramount. Safeguard that information by leveraging encrypted DNS across our platforms to deliver private and secure connectivity within your app. Discover how you can use system DNS settings to connect to encrypted servers or enable encrypted DNS within an app using standard networking APIs.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10047", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



DNS is short for Domain Name Server, it converts the webpage address you’re writing in the browser to actual IP addresses of servers on the internet, these questions are usually unencrypted and can be listened to or even interfered with.

New in all Apple Platforms the support for encrypted DNS:

* `DoT`: DNS over TLS
* `DoH`: DNS over HTTPS

Both uses TLS to encrypt messages, but `DoH` additionally uses HTTP to improve performance.
You can configure encrypted DNS in two ways:

## 1. System-wide configurations:

* Selects a single DNS server as the single resolver for all apps on the system
* This can be done by writing a network extension app (or using a profile in case of MDM for enterprises) that configures the system to only use this single DNS server
* This is an example of creating a network extension app to add a system-wide DNS configuration:

```swift
// Create a DNS configuration

import NetworkExtension

NEDNSSettingsManager.shared().loadFromPreferences { loadError in
	if let loadError = loadError {
    	// ...handle error...
    	return
	}
	let dohSettings = NEDNSOverHTTPSSettings(servers: [ "2001:db8::2" ])
	dohSettings.serverURL = URL(string: "https://dnsserver.example.net/dns-query")
	NEDNSSettingsManager.shared().dnsSettings = dohSettings
	NEDNSSettingsManager.shared().saveToPreferences { saveError in
    	if let saveError = saveError {
        	// ...handle error...
        	return
    	}
	}
}
```

* Some compatibility rules that are applied by default when using a system-wide DNS settings:
	* Captive network detection (like when someone logs into a café network) is automatically granted an exception from your DNS configuration
	* When using VPNs, the DNS settings within the VPN tunnel is used over the configured DNS system settings

* However, there are other rules you will have to setup yourself. For example, here is how you’d define a rule if you want to only grant an exception to a specific domain when using the work WiFi network:

```swift
// Apply network rules

let workWiFi = NEOnDemandRuleEvaluateConnection()
workWiFi.interfaceTypeMatch = .wiFi
workWiFi.ssidMatch = ["MyWorkWiFi"]
workWiFi.connectionRules =
	[ NEEvaluateConnectionRule(matchDomains: ["enterprise.example.net"],
                    		   andAction: .neverConnect) ]

let disableOnCell = NEOnDemandRuleDisconnect()
disableOnCell.interfaceTypeMatch = .cellular

let enableByDefault = NEOnDemandRuleConnect()

NEDNSSettingsManager.shared().onDemandRules = [
	workWiFi,
	disableOnCell,
	enableByDefault
]
```

> There is a demo that shows a sample project on how you can fully implement a system-wide encrypted DNS

## 2. Application opt-in configurations:

* You can configure your app only to use a single DNS resolver for some all of your app connections
* This works regardless of the way your app is accessing the network, `URLSession` tasks, `Network.framework` connections, or even POSIX APIs like `getaddrinfo`
* This is an example of how you’d do it with `Network.framework`:

```swift
// Use encrypted DNS with NWConnection

import Network

let privacyContext = NWParameters.PrivacyContext(description: "EncryptedDNS")
if let url = URL(string: "https://dnsserver.example.net/dns-query") {
	let address = NWEndpoint.hostPort(host: "2001:db8::2", port: 443)
	privacyContext.requireEncryptedNameResolution(true,
    	fallbackResolver: .https(url, serverAddresses: [ address ]))
}

let tlsParams = NWParameters.tls
tlsParams.setPrivacyContext(privacyContext)

let conn = NWConnection(host: "www.example.com", port: 443, using: tlsParams)
conn.start(queue: .main)
```

* Here is how to opt-in all of your application connections using other APIs:

```swift
// Use encrypted DNS for other APIs

import Network

if let url = URL(string: "https://dnsserver.example.net/dns-query") {
	let address = NWEndpoint.hostPort(host: "2001:db8::2", port: 443)
	NWParameters.PrivacyContext.default.requireEncryptedNameResolution(true,
    	fallbackResolver: .https(url, serverAddresses: [ address ]))
}

let task = URLSession.shared.dataTask(with: ...)
task.resume()

getaddrinfo(...)  
```

> Note: System-wide DNS configurations will take precedence over application specific ones

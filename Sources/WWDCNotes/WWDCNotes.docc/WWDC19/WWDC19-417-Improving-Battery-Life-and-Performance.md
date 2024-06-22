# Improving Battery Life and Performance

Learn about new ways to find and fix performance issues during daily development, beta testing, and public release on the App Store. Learn how to catch performance issues during daily development by measuring CPU, memory, and more in your XCTests. Discover how to find issues in the field during beta testing and public release using MetricKit. See how the Xcode Organizer now displays the most important metrics from your app aggregated from each version on the App Store.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/417", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## New Tools

### XCTest Metrics

Performance and battery usage directly in tests (analyzing blocks)

### [MetricKit][metricKitDocs]

On device framework that anonymously collects metrics about your app performance and battery usage on real users, we can decide what to measure.

### Xcode Metrics Organizer

In Organizer we have a new tab showcasing Metrics (anonymously collected by Apple)


## Battery Metrics

- Processing (CPU usage) 
- Location 
- Display 
- Networking 
- Accessories (Bluetooth)
- Multimedia 
- Camera 

## Performance Metrics

- Hangs (histogram on when the app remains unresponsive to user input)
- Disk 
- Application Launch
- Memory
- Custom Intervals

With MetricKit we can add tests and put baselines on those tests: if a test passes that baseline, it automatically fails.

## Custom MetricKit Reports

MetricKit reports will be sent to a delegate (in our app), which will be responsible to send the data to our servers.

Our MetricKit delegate will be called up to once per day with a MXMetricPayload, which then we can upload. We can simulate a payload in the simulator: “Debug > Simulate MetricKit Payload”

Standard MetricKit Reports
Apple will also gather metrics and report them anonymously to us like it does with Energy consumption and crashes via the Organizer. (this is automatic, no need for any change in our app)

[metricKitDocs]: https://developer.apple.com/documentation/metrickit
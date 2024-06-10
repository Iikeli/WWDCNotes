# iPad and iPhone apps on Apple silicon Macs

Apple silicon Macs can run many iPad and iPhone apps as-is, and these apps will be made available to users on the Mac through the Mac App Store. Discover how iPad and iPhone apps run on Apple silicon Macs, and the factors that make your apps come across better. Learn how to test your app for the Mac, and hear about your options for distribution of your apps.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10114", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Compatibility

For your app to be available in the Mac App Store, it must be compatible with the Mac:

- Your app can't be dependent on an unavailable symbols or frameworks
- It can't be dependent on missing functionality in existing frameworks
- It can't be dependent on hardware capabilities that don't exist on the Mac

## Availability

- All iOS/iPadOS apps are automatically available in the Mac app store
- You can opt out in App Store Connect

## Environment differences

### Hardware differences

- Mouse and touch events: the system automatically maps standard gestures, however you should test if your custom interactions and gestures work on Mac
- Environment sensors: make sure to check for the availability of each sensor in your app 
- Camera: to discover all available cameras (you can plug external cameras on the mac) use [`AVCaptureDeviceDiscoverySession`][AVCaptureDeviceDiscoverySession]

### UI differences

- Popups might be displayed in different places in macOS
- Open and Save panels will show in separate window
- If your iPad app already supports multitasking on iOS, it will be fully resizeable on macOS. (if not, your app will be displayed in a fixed-size window)

### System Software differences

- Filesytem: On macOS the user can move the app wherever they would like. Your app's data containers are also in a different location on the Mac. Use Foundation APIs to locate items in the filesystem.

## Distribution

- Beside the mac app store, you can also distribute your apps via ad-hoc and other methods as well
- App thinning is supported
- TestFlight is not supported on macOS

[AVCaptureDeviceDiscoverySession]: https://developer.apple.com/documentation/avfoundation/avcapturedevicediscoverysession
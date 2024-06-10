# Get to know Developer Mode

Meet Developer Mode — required on iOS 16, iPadOS 16, and watchOS 9 to install, run, and debug your apps during development. We'll show you how you to opt in to Developer Mode on your devices, and how to enable Developer Mode in your automation workflows.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110344", purpose: link, label: "Watch Video (5 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What is Developer Mode

- new mode in iOS 16 and watchOS 9 that enables common developer workflows
- disabled by default, requires you to explicitly enroll the device into Developer Mode
- enrollment persists across reboots and system updates
- setup can be automated
- only required for local development
- Most common distribution and testing flows **do not** require Developer Mode:
  - Test Flight
  - Enterprise (In-House) distribution
  - App Store

### Why Developer Mode?

- Developer features are being abused in targeted attacks
- Most people do not need to use developer feature by default

## Using Developer Mode (when and how to use)

### When to turn on Developer Mode

Turn on developer mode if you need to:

- Run and install Development signed applications
  - Includes applications signed using a Personal Team

- Debug and instrument your applications
- Automate testing

### How to turn on Developer Mode

- Requires to connect your device to Xcode for the Developer Mode menu item to appear on the iPhone
  - iOS 16 beta releases will have the menu item always visible for the time being

- Developer Mode controls are in Settings.app/Privacy & Security

> Turning on Developer Mode requires that you reboot your device

Once the reboot is completed, you can run your app onto your device

## Automation flows

- Only devices without passcodes can be automatically enrolled into Developer Mode

- macOS Ventura ships with `devmodectl` that can enable Developer mode for:
  - a single device that you have already connected (Single device one-off mode)
  - all devices that you plug in (Streaming mode)

To change/enable `devmodectl` mode, run `devmodectl` in your cli:

```shell
$ devmodectl streaming
```

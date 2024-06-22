# Optimizing App Launch

Slow app launches are frustrating. Learn about the new app launch instrument and discover how to make your app launch fast. Gain insights into what happens during app launch and how to minimize, prioritize, and optimize work at this critical time. Hear tips and tricks from the engineers making iOS apps launch fast.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/423", purpose: link, label: "Watch Video (43 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Launch phases

1. System InterfaceSeparate in two phases:
  - DYLD3Dynamic Linker: loads shared libraries and frameworks
  - libSystem Init: initializes app low level components

2. Static Runtime init
  - Separate in two phases:
    - Initializes the language runtime (objc/swift)
    - Expose dedicate initialization API in frameworks.
    - Reduce impact to launch by avoiding `+[Class load]`
    - Use `[Class initialize]` to lazily conduct static initialization

3. UIKit init
  - This is when the system initializes `UIApplication` and `UIApplicationDelegate`

4. Application init
  - Invokes `UIApplicationDelegate` app lifecycle callbacks `UIApplicationWillFinishLaunching` / `didFinish` / `DidBecomeActive`
  - (iOS 13 and later) We now get UISceneDelegate UI lifecycle callbacks for each scene: `willConnectToSession:`, `sceneWillEnterForeground:`, `sceneDidBecomeActive`.

5. Initial Frame Render
  - this is when the app views are created, their layout is performed, and they’re drawn. `loadView` `viewDidLoad` `layoutSubviews`
  - tips:
    - reduce the number of views launched at the beginning
    - flat view hierarchies and lazily load views not needed during launch

6. Extended
  - apps specific logic

## How to test our app launch time

We have an instrument entirely dedicated to analyze launch times: this will split the app launch in all the sections seen above and highlight our code that executes during this phase.

The new XCTest Metrics is also a very useful and lightweight way to make sure to not incur in some huge slowdowns during development.

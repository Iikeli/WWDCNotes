# Architecting Your App for Multiple Windows

Dive into the details about what it means to support multitasking in iOS 13. Understand how previous best practices fit together with new ideas. Learn the nuances of structuring your application to support multiple windows, and how to instantiate your UI, handle windows coming and going, and manage your app’s underlying window resources.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/258", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



In the new scene delegate based lifecycle:

- app delegate should still handle things like initializing/cleaning up the whole process
- it should not however handle things like setting up the UI or updating it when the app goes into foreground/background - because this will now happen separately per session
- additionally, application delegate is now notified when a scene is created or discarded
- lifecycle delegate methods like [`applicationWillEnterForeground`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623076-applicationwillenterforeground), [`applicationDidBecomeActive`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622956-applicationdidbecomeactive) etc. will not be called at all if scene lifecycle is enabled

## Modern lifecycle flow

Starting the app:

- `AppDelegate`: [`didFinishLaunchingWithOptions:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622921-application)
- `AppDelegate`: [`_:configurationForConnecting:options:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197905-application) `->` [`UISceneConfiguration`](https://developer.apple.com/documentation/uikit/uisceneconfiguration) - return configuration for the scene
- `SceneDelegate`: [`_:willConnectTo:options:`](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197914-scene) - this is where you set up your new scene

Scene configuration ([`UISceneConfiguration`](https://developer.apple.com/documentation/uikit/uisceneconfiguration)): an object that describes the delegate class, storyboard etc. It's either built dynamically, or (better) statically described in `Info.plist` and looked up by name using the constructor:

```swift
UISceneConfiguration(name: "Default", sessionRole: session.role)
```

Going to the background:

- `SceneDelegate`: [`sceneWillResignActive`](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197919-scenewillresignactive)
- `SceneDelegate`: [`sceneDidEnterBackground`](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197917-scenedidenterbackground)
- …
- `SceneDelegate`: [`sceneDidDisconnect`](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197916-scenediddisconnect) - after some time, the system will release your scene (including the delegate object and all views and VCs) to save memory; you should now release any objects loaded to build that piece of UI

When the user force-closes a scene:

- `AppDelegate`: [`_:didDiscardSceneSessions:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197906-application) (may be called on the next launch instead)

> *“State restoration is no longer a nicety”*

Watch out for cases where a view controller needs to be updated if there are multiple scenes presenting the same view, which you might not have taken into account before - it’s better to only update the model and observe changes in the model, instead of updating the view directly when the user adds some new content.

- pro tip: you can pass Swift-only objects like enums in [`NSNotification`](https://developer.apple.com/documentation/foundation/nsnotification) if you use them as the sending object instead of in `userInfo`

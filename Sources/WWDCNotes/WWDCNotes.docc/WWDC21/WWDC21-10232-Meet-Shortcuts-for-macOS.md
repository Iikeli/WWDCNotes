# Meet Shortcuts for macOS

Shortcuts is coming to macOS, and your apps are a key part of that process. Discover how you can elevate the capabilities of your app by exposing those features as Shortcuts actions. We’ll show you how to build actions for your macOS apps built with Catalyst or AppKit, deploy actions across platforms, publish and share shortcuts, and enable your app to run shortcuts from other apps. We’ll also take you through how Shortcuts fits in with existing Mac automation technologies like Automator and AppleScript.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10232", purpose: link, label: "Watch Video (26 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



- All features available from iOS app
- Shortcuts build editor rewritten
- App mostly written in SwiftUI
- A lot of new actions added to support Mac-specific needs
- Lots of FileProvider actions automatically supported for apps using FileProvider
- Shortcuts get their own file format & can be easily shared (e.g. on a website)
- Downloaded shortcuts ask for permission when accessing data libraries
- Shortcuts are signed or notarized for improved security
- Full support for AppleScript via new action
- Migration Tool that can convert most Automator workflows to Shortcuts
- Third party apps can expose actions from the app, system will make use in many places
- Data can be thought of as “Nouns”, the actions as “Verbs” – one action per verb
- Actions defined via SiriKit Intent Definition files, each action is an intent
- Minimum requirement is a `displayString` and `identifier` property – more possible
- Xcode creates classes to use in app, similar to generated Core Data model code
- App will need to handle incoming intents:
    - Either handle them in-app: Most apps should start with this
    - Or handle via Intent Extension: Lightweight standalone process

- AppDelegate methods `application(handlerFor: Intent)` needs to be implemented
- Implement an IntentHandler conforming to `<ActionName>IntentHandling`
- Intent definition file supports custom validation errors that can be "thrown" to the system
- Contributor comment: *Building an intent has 6 steps involved, it’s a lot of work, but worth it for pro users*
- Mac Catalyst apps intents automatically work on Mac, too
- Same exact intent definition can be used in both iOS & Mac apps for shared workflows
- MacOS has a command line tool called `shortcuts`

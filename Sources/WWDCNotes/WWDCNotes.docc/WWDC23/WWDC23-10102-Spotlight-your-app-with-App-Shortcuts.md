# Spotlight your app with App Shortcuts

Discover how to use App Shortcuts to surface frequently used features from your app in Spotlight or through Siri. Find out how to configure search results for your app and learn best practices for creating great App Shortcuts. We’ll also show you how to build great visual and voice experiences and extend to other Apple devices like Apple Watch and HomePod.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10102", purpose: link, label: "Watch Video (25 min)")

   @Contributors {
      @GitHubUser(srujanc)
   }
}



# App Intents

Extend your app’s custom functionality to support system-level services like Siri, Spotlight, the Shortcuts app, and the Action button using [App Intents](https://developer.apple.com/documentation/AppIntents).

The App Intents framework offers a programmatic way to make your app’s content and functionality available to system services like Siri and the Shortcuts app. The programmatic approach lets you expose any of your app’s capabilities, and not just ones that fall into specific categories. You also use this programmatic approach to supply metadata, UI information, activation phrases, and other information the system needs to initiate your app’s actions.


### App Actions

***App intents ***

    Define the custom actions your app exposes to the system, and incorporate support for existing SiriKit intents.

***Parameter resolution***
    
    Define the required parameters for your app intents and specify how to resolve those parameters at runtime.

***Resolvers***
    
    Resolve the parameters of your app intents, and extend the standard resolution types to include your app’s custom types.

### Data introspection

***App entities***

    Make core types or concepts discoverable to the system by declaring them as app entities.

***Entity queries***

    Help the system find the entities your app defines and use them to resolve parameters.

### System integration

***App Shortcuts***

    Integrate your app’s intents and entities with the Shortcuts app, Siri, Spotlight, and the Action button on supported iPhone and Apple Watch models.

***Intent discovery***

    Donate your app’s intents to the system to help it identify trends and predict future behaviors.

***Focus***
    
    Adjust your app’s behavior and filter incoming notifications when the current Focus changes.

***Action button on iPhone and Apple Watch***
    
    Enable people to run your App Shortcuts with the Action button on iPhone or to start your app’s workout or dive sessions using the Action button on Apple Watch.

### Utility types

***Common types***
    
    Specify common types that your app supports, including currencies, files, and contacts.



### App Shortcuts

* App Shortcuts are built with the App Intents framework, the Swift only framework built from the ground up to make it faster and easier to build great intents right in the Swift source code. All App Shortcuts begin by defining an intent in the source code. 

* Intents represent individual tasks that can be completed with the app, like creating a to-do list, summarizing its contents, or checking off an item. 

After creating an app intent, create an app shortcut with it, so it can be used from Spotlight or Siri. This associates the app intent with Siri trigger phrases, titles, and symbols that are needed.


``` swift

struct DemoAppShortcutsProvider: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut (
            intent: CreateList(),
            phrasos: ["Create a new \(copplicationName) list"],
            shorttitio: "Creato List",
            systemimageName: "checklist"
        )
    }
}

```

By running the app you can immediately start creating to-do lists right from Siri or the Shortcuts app, all by only creating two structs in the code.

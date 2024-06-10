# Design for intelligence: Make friends with "The System"

The building blocks of the intelligent system are simple: Define, learn, execute. Discover how you can use intents to define your app's key features, create donations to help the system learn and make predictions about the future, and implement extensibility to ensure your app is ready to execute at just the right moment. Learn from teams at Apple about how their technologies use intents and donations in different ways, all for the same goal: to make the everyday easier.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10087", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- The system intelligence goal is to make users everyday interactions as smooth and frictionless as possible
- To make this happen, your app and the system need to help each other: this is done via the `Intents.framework`

## Intents Key concepts

### Define (Intents)

As developers and designers we need to ask ourselves what are the important actions that people want to perform with our app.

The goal is to determine what are the key and repeatable things that people want to accomplish: we can define these actions using Intents.

Intents represent what people do in the app.

The intents you define are a key building blocks that enable your app and the system to speak the same language.

A key piece of the intelligence system experience is learning how people do things in order to predict how they might do so again in the future.

### Learn (Donations)

When the user executes an intent, the app can _donate_ such intent to the system: these donations provide signals that the system can use to learn from.

The system will eventually pick up a pattern and start suggesting those intents to the user as Siri Suggestions, which surface on the lock screen.

A Donation is a record, a snapshot in time of an intent that's actually been executed.

Time and location are only a few of the signals that the system uses to understand when something is important, another important one is **context** (for example leaving for work).

### Execute (handle actions)

It is important for our app to be ready to perform an action.

Once the user taps on an Intent (from Siri Suggestions), there are two ways our app can handle it:

- via background execution (preferred)
- via deeplink

## How to use these technologies

### Shortcuts

If the user does over and over the same action, they shouldn't need to open the app everytime, setting up a shortcut is preferred: the way we setup shortcuts is by defining intents

### Widgets

There are some pieces of information that are so important the user want to see them time and time again: this is the problem Apple wants to solve with the new and improved widgets.

### Siri Event Suggestions

Siri event suggestions help you display the user events/reservations etc at the right place and time.

With iOS 14 and macOS Big Sur, there's a new way to integrate events with the system also via Mail and Safari by using web markup.

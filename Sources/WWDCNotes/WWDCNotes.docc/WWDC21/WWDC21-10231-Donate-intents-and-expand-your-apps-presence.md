# Donate intents and expand your app’s presence

Discover how you can make key parts of your app available for someone at exactly the right moment — without them ever needing to open it. Learn how to craft and donate intents to the system, helping you surface relevant and contextual information about your app in Siri, Focus, Shortcuts, the Smart Stack, and more. We’ll explore how the system intelligently identifies information and show you techniques for structuring intents to help increase engagement and visibility for your app.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10231", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What are intents?

- a way of representing, in detail, a type of action that your app can perform
- Siri includes a lot of built-in intents that support a range of capabilities that you can use to integrate with the system (e.g., messaging, ride booking, payments)
- support the ability for you to define your own custom intents for use with Shortcuts

## What are intent donations?

- An intent donation is the act of telling the system when a person performs an action in your app
- The system will store these donations and use them to expand your app's capabilities and presence throughout the system
- Apps that donate intents show up in places throughout the system like Suggestions, Shortcuts, Focus, Smart Stack, and Siri

## The life of an intent donation

1. Create a new intent definition:

  - With your project open, create a new file of type <kbd>SiriKit Intent Definition</kbd> File
  - in the following menu select any targets that I want to use the custom intent

2. Define the intent
  - After selecting the new Intent file, click the <kbd>+</kbd> button on the bottom right and select <kbd>New Intent</kbd> (or <kbd>Customize System Intent</kbd> if your intent matches one of the existing built-in system intents)

3. Define eligibility
  - After naming your intent in step two, check whether you'd like your intent to be eligible for widgets/shortcuts/(Siri)suggestions

4. Define Intent parameters
  - These parameters are the input of your action/intent, which depend on your specific intent
  - For each parameter you can select the type (String/Location/URL... can be custom, too)
  - If the parameter options are not static and/or need to be provided by the app, check the <kbd>Dynamic Options</kbd> - this 

The system reads the intent definition file to determine the intent and parameter combinations that an app supports for prediction.

5. Donate the intent when appropriate
  - Create an instance of the intent class (this is automatically generated for you from the Intent file via an Xcode rule)
  - donate it

```swift
// Donate your intent.

let intent = CheckWeatherIntent()
intent.location = weatherLocation

let interaction = INInteraction(intent: intent, response: nil)
interaction.donate { (error) in
  // Handle the error.
}
```

- It's important to delete donations so people don't get suggestions for actions in your app that are no longer relevant:

```swift
// Donate your intent.
let interaction = INInteraction(intent: intent, response: response)
interaction.identifier = "68753A44-4D6F-1226-9C60-0050E4C00067"
interaction.groupIdentifier = "san-diego"
interaction.donate { (error) in
  // Handle the error.
}

// Delete individual donations.
INInteraction.delete(with: ["68753A44-4D6F-1226-9C60-0050E4C00067"]) { (error) in
  // Handle the error.
}

// Delete group donations.
INInteraction.delete(with: "san-diego") { (error) in
  // Handle the error.
}
```

## Behind the scenes

- the system stores these donations over time along with the context the person was in when the intent was donated
- As the user continue to repeat the same action, the app continues to make intent donations
- The system uses the intent parameters to determine whether the intents are equivalent or not
- The system uses machine learning and on-device intelligence to find patterns in the data and predict what intent is relevant given a person's current context and past behavioral patterns
- The on-device intelligence integrates with user-facing features to expose your app's capabilities and presence to the user
- all of the machine learning and intelligence is performed on-device in a privacy-preserving manner, meaning Apple does not collect any data that can be used to personally identify a user

## What makes a great donation?

- Represents an action likely to be repeated by the user
- Intent payload is consistent across donations - so that a pattern can be recognized by the on-device intelligence 
- No timestamps (in the suggestion parameters)
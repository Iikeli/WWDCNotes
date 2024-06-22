# Introducing Parameters for Shortcuts

Parameters take Siri Shortcuts to the next level, enabling an interactive voice experience in Siri with follow-up questions, and allowing people to customize shortcuts in the Shortcuts app, now built into iOS. Walk through setting up your shortcuts to take advantage of parameters and learn how your shortcuts can pass output to other actions when creating multi-step shortcuts in the Shortcuts app.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/213", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- Shortcuts can be used via `Suggestions` (at the right time), `Voice` (to use any capabilities of your app just by asking Siri) and `Multi Step Shortcuts` (by composing multiple shortcuts in a new one)
- Support for conversational shortcuts which lets users customize your Shortcut behavior
- Shortcuts app redesigned and build in to iOS
  - Adds `Automation` tab which lets users run shortcuts on certain conditions (Sunset, NFC-Tag, Before I leave for work, ...)
  - Gallery with hundreds of pre-build Shortcuts - also from 3rd party apps
  - 3rd party Shortcuts can provide outputs and thus be connected to other Shortcuts

- Let users discover Shortcut capabilities by showing the `Add to Siri` button at appropriate stages
- Redesigned `Add to Siri` sheet where users can enter the activation phrase and tweak the Shortcut parameters
- **Shortcut Customization**
  - In the **Intent Editor** you can 
    - Create Shortcuts your app offers to your users
    - Customize an intents parameters even with custom types (in iOS 13)
    - Control if parameters are exposed to the Shortcuts app

- **Parameter Resolution**
  - `Resolve` (prior `Confirm` and `Handle`) as newly added Shortcut lifecycle step
  - Siri goes through each parameter until it knows what to fill in for each
  - After resolution Siri invokes the same confirm and handle methods as in iOS 12
  - Resolution result can be: `.needsValue`, `.disambiguation`, `.unsupported`, `.confirmationRequired`, `.success`, `.notRequired`
    - User-facing message for each of them can be customized in intent editor

- **Related Parameters**
  - Express parameter relationships in the `Intent Editor > Section "Shortcuts App"`
  - Developers have to
    - Identify parent and child parameters
    - Establish parameter relationships
    - Update summaries for each parameter combination

- **Dynamic Options**
  - Check the Dynamic Options checkbox in Intent Editor
  - Provide the supported values and default values from delegate functions

- **Outputs**
  - Specify your own type that contains the properties you want to pass on to other actions 
  - Define a new property of your output type and designate it to be an output

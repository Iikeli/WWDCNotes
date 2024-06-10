# Bring widgets to life

Learn how to make animated and interactive widgets for your apps and games. We’ll show you how to tweak animations for entry transitions and add interactivity using SwiftUI Button and Toggle so that you can create powerful moments right from the Home Screen and Lock Screen.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10028", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(joforsell)
   }
}



## Key takeaways

#### Changes between entries can now be animated.
#### Widgets can now be interactive.

<br></br>

## Animations

- Changes between entries default to an implicit spring animation and various implicit content transitions
- Animations can be customized with the same APIs you use to customize SwiftUI views; `.animation`, `.transition` and `.contentTransition`.
- The new #Preview macro can be used to quickly preview animations between entries.

```Swift
#Preview(as: WidgetFamily.systemSmall) {
    CaffeineTrackerWidget()
} timeline: {
    CaffeineLogEntry.log1
    CaffeineLogEntry.log2
    CaffeineLogEntry.log3
    CaffeineLogEntry.log4
}
```

<br></br>

## Interactivity

- Actions are exposed to your widget using AppIntents
- AppIntents are passed through new initializers on `Button` and `Toggle`. These are the two only controls that work with widgets so far.

```Swift
extension Button {
    public init<I: AppIntent>(
        intent: I,
        @ViewBuilder label: () -> Label
    )
}

extension Toggle {
    public init<I: AppIntent>(
        isOn: Bool,
        intent: I,
        @ViewBuilder label: () -> Label
    )
}
```

- The return of an AppIntents perform() method will trigger a reload of the widgets UI, so make sure to make any changes to the state before that.
- Use the `@Parameter` property wrapper to pass properties to your AppIntent.
- You can choose the widget extension as the build target, then Xcode will install the widget for you.
- Use the `.invalidatableContent` modifier to indicate a view will update before the new value has arrived, to reduce the feeling of latency.

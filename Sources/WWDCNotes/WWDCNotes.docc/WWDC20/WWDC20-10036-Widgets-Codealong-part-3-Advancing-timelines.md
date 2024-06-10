# Widgets Code-along, part 3: Advancing timelines

Take your widget to the next level as we embark upon the third and final stage of the widgets code-along. Pick up where you left off in Part 2 or start with the Part 3 starter project to go warp speed ahead. We’ll explore advanced concepts for widgets, timelines, and configuration. Learn how to load in-process and background URLs and link directly to content within your app. And discover how to create multiple widgets that explore different features within your app, as well as making your widget dynamically configurable.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10036", purpose: link, label: "Watch Video (9 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## URL Sessions

- the `TimelineProvider` API is built with completion handlers instead of return values, specifically to make doing asynchronous tasks, like network fetches, easy
- we can use normal URL sessions end even background session tasks.
- since the widget has no app delegate, there's a new `View` modifier, introduced just for widgets, to handle background URL sessions: [`.onBackgroundURLSessionEvents(matching:_:)`][bgSessionDoc]

## Link

- use SwiftUI's [`Link`][linkDoc] to make elements of the widget deep link into your app.
- `Link` behaves like `Button`, but instead of passing an `action`, we pass a `destination` `URL` which will be passed to the app on launch.
- to read the URL in your views, use the [`.onOpenURL(perform:)`][onOpenDoc] modifier

## Widget bundles

- In order to create/support multiple widgets in the same extension, we need to define a [`WidgetBundle`][wbDoc]
- The `WidgetBundle` will be the new entry point of the extension
- The `WidgetBundle` declares all available widgets in its body (up to 5 widgets)

```swift
@main
struct EmojiRangerBundle: WidgetBundle {
    @WidgetBundleBuilder
    var body: some Widget {
        EmojiRangerWidget()
        LeaderboardWidget()
    }
}
```

## Dynamic configuration

- if we don't know all of the configuration options at build time, we can provide a dynamic list of options via an SiriKit `Intent` extension, the same way we would for other Intent-based features
- to create a SiriKit extension, go to `File > New Target` and choose `Intents Extension`.
- in our `Intent` editor we will set our Intent type to something custom (in the screenshot above we set it to `Hero`):

![][customTypeImage]

- if we look at the definition of our custom type, by default it has two properties: `identifier` and `displayString`:

![][customTypeImage2]

- these properties are defined in our Intent extension:

```swift
class IntentHandler: INExtension, DynamicCharacterSelectionIntentHandling {
    
    func provideHeroOptionsCollection(for intent: DynamicCharacterSelectionIntent,
                                      with completion: @escaping (INObjectCollection<Hero>?, Error?) -> Void) {
        let characters: [Hero] = CharacterDetail.availableCharacters.map { character in
            let hero = Hero(identifier: character.name, display: character.name)
            
            return hero
        }
        
        let remoteCharacters: [Hero] = CharacterDetail.remoteCharacters.map { character in
            let hero = Hero(identifier: character.name, display: character.name)
            
            return hero
        }
        
        let collection = INObjectCollection(items: characters + remoteCharacters)
        
        completion(collection, nil)
    }
    
    override func handler(for intent: INIntent) -> Any {
        return self
    }
}
```

- lastly, we will need to update our `IntentTimelineProvider` to support our dynamic configuration

[customTypeImage]: WWDC20-10036-custom-type
[customTypeImage2]: WWDC20-10036-custom-type2


[bgSessionDoc]: https://developer.apple.com/documentation/widgetkit/staticconfiguration/onbackgroundurlsessionevents(matching:_:)-2c8e7
[linkDoc]: https://developer.apple.com/documentation/swiftui/link
[onOpenDoc]: https://developer.apple.com/documentation/swiftui/group/onopenurl(perform:)?changes=latest_minor
[wbDoc]: https://developer.apple.com/documentation/swiftui/widgetbundle
# Bring widgets to new places

The widget ecosystem is expanding: Discover how you can use the latest WidgetKit APIs to make your widget look great everywhere. We’ll show you how to identify your widget’s background, adjust layout dynamically, and prepare colors for vibrant rendering so that your widget can sit seamlessly in any environment.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10027", purpose: link, label: "Watch Video (7 min)")

   @Contributors {
      @GitHubUser(arnoappenzeller)
      @GitHubUser(mikakruschel)
   }
}



## Widgets in 4 new locations

- Desktop on Mac, Lock screen on iPad, StandBy mode on iPhone, Smart Stack on Apple Watch
- Widgets can appear in all these places automatically
- iOS Widgets can be used on Mac without Mac App

## New content margins

- Automatically added padding to prevent the content to go to close to the edge of the widget container
- Replaces safe areas used on watchOS 9 and below
- To allow content to go to the edge of the widget container, add the `.contentMarginsDisabled()` modifier to the widget configuration
  - For content that should stay in the default content margins, use the `widgetContentMargins` environment and add `.padding(margins)` back in

```swift
struct SafeAreasWidgetView: View {
    @Environment(\.widgetContentMargins) var margins

    var body: some View {
        ZStack {
            Color.blue
            Group {
                Color.lightBlue
                Text("Hello, world!")
            }
            .padding(margins) 
        }
    }
}

struct SafeAreasWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(...) {_ in
            SafeAreasWidgetView()
        }
        .contentMarginsDisabled()
    }
}
```

## Removable background

- Use `.containerBackground` modifier
  - System automatically takes out the widget's background depending on where it's being shown (e.g. iPad Lock screen)
  - Smart Stack on Apple Watch also uses the `containerBackground` instead of the default a dark material background
- Add `.containerBackgroundRemovable(false)`  to widget configuration to prevent background removal
  - This makes the widget ineligible in various contexts that require a removable background (e.g. StandBy on iPhone or iPad Lock screen)

## Dynamically adjust layout

- Layout should adopt for when the background is removed by pushing the content to the edges (happens automatically with content margins) and enlarging important elements (like current temperature in the weather widget)
- Make it dependent on environment variable  `showsWidgetContainerBackground`

## Vibrant rendering mode

- Vibrant rendering mode automatically applied on iPad Lock screen and StandBy Night mode
  - Widgets will be desaturated and colored appropriately for the Lock screen background
  - This could remove contrast and impact the widget's legibility
- Detect with `widgetRenderingMode` environment variable

# Inspectors in SwiftUI: Discover the details

Meet Inspectors — a structural API that can help bring a new level of detail to your apps. We’ll take you through the fundamentals of the API and show you how to adopt it. Learn about the latest updates to sheet presentation customizations and find out how you can combine the two to create perfect presentation experiences.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10161", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(kslazinski)
   }
}



## Inspector

- It's a new element in SwiftUI.
- Inspector is a view that shows details of selected content.
- Inspector is available to SwiftUI Developers on macOS, iPadOS and iOS.
- Users are familiar with inspectors as they are already present in many apps, for example Keynote or Shortcuts.

![Keynote app example][keynote]

[keynote]: WWDC23-10161-keynote

Inspector is an addition to other SwiftUI Structural APIs like: NavigationSplitView, NavigationStack, .sheet, .popover, .alert, .confirmationDialog

![SwiftUI Structural APIs][structuralAPIs]

[structuralAPIs]: WWDC23-10161-structuralAPIs

The inspector API includes programmatic control over:
- column width
- presented state (allowing hiding and showing of the inspector as needed)

In compact size classes, inspector adapts to a resizable sheet and inspector will automatically overlay in split screen on larger iPads.

## How to

To add an inspector to a SwiftUI view, we use .inspector modifier.
Example:
```
.inspector(isPresented: $presented) {
  InspectorForm(item: $state.binding())
}
```

Inspectors can collapse by default, but they aren't resizable by default. We can change it with .inspectorColumnWidth. We can also add a toolbar button to toggle the "presented" property.
Example:
```
.inspector(isPresented: $presented) {
  InspectorForm(item: $state.binding())
    .inspectorColumnWidth(min: 200, ideal: 300, max: 400)
    .toolbar {
      Spacer()
      Button{
        presented.toggle()
      } label: {
        Label("Toggle Inspector", systemImage: "info.circle")
      }
    }
}
```

![Toggle Button][toggleButton]

[toggleButton]: WWDC23-10161-toggleButton

## Placement

Context and placemend of the .inspector modifier and its .toolbar matters. Inspector can be placed inside or outside of a navigation structure like a NavigationStack or NavigationSplitView. Inspector's toolbar content be inside or outside of the inspector's view builder. Based on these factors we can achieve few different results:

![Inspector placement][placement]

[placement]: WWDC23-10161-placement

If you are using an inspector within a NavigationSplitView, the inspector should be placed in the detail column's view builder, or it can also be placed entirely outside the navigation structure.

## Presentation customizations in iOS 16.4

Apart from new .inspector modifier, SwiftUI offers now more customization of sheets and other presentations like popovers.

- .presentationBackground (sets a background of a presentation)
- .presentationBackgroundInteraction (can enable interaction with the content behind the presentation with .enabled argument)
- .presentationCornerRadius
- .presentationContentInteraction
- .presentationCompactAdaptation

These modifiers also work with Inspector when Inspector is presenting as a sheet.

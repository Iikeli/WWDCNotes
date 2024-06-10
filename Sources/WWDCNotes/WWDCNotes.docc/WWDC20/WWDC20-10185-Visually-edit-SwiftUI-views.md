# Visually edit SwiftUI views

Help your apps be the best versions of themselves: Discover how you can leverage Xcode Previews and SwiftUI to quickly iterate upon and improve your app. Find out how you can use the Previews canvas to build your app from the ground up, and view your interface in different environments like Light or Dark mode or with accessibility features like Dynamic Type enabled.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10185", purpose: link, label: "Watch Video (5 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



## Overview
This session is about using Xcode Previews to quickly build SwiftUI apps. In this session, we'll be building the row view for a smoothie app.

The view has:

- Smoothie name
- Ingredients
- Calories
- Popularity rating
- Picture

![][row_view_outline]

## Adding View Elements
To start, we use the library to add SwiftUI components by dragging and dropping onto the Preview.

![][drag_and_drop]

SwiftUI uses adaptive layout to ensure our app looks good whether it's running on a Mac or iPhone. It also updates our code automatically.

Before, we had:

```swift
var body: some View {
  Text("Hello World")
}
```

After dropping, the code automatically changes to:

```swift
var body: some View {
  VStack {
    Text("Hello World")
    Text("Placeholder")
  }
}
```

## Editing Views
If we want to modify any part of our view, like a label or image, we can double click to automatically be taken to the corresponding SwiftUI code. The preview will immediately update as the source code changes.

![][live_update]

This extends to variables as well:

![][live_update_variables]

**We can also use CMD+D to duplicate views.** If you duplicate a view that isn't in a container, Xcode will insert one for you.

## Images and Modifiers
We can use CMD+Shift+L to quickly bring up the view library. The library also allows us to use our Asset catalog media and directly drop that into the Preview. This will also automatically generate the relevant SwiftUI code in the editor.

![][custom_media]

When we drop the image, it is much larger than expected. This is because SwiftUI renders images in their true size.

![][large_image]

To remedy this, we can click on the image and use the inspector on the right. In the inspector, we can click "Add Modifier", which shows a dropdown of recommended suggestions for the current context.

![][add_modifier]

We can use this to try different modifiers and see what the results look like without having to re-launch our app every time. We also don't need to know what the modifier's signature needs to be in code when using the dropdown menu.

We get this code without having to write any of it:

```swift
var body: some View {
  HStack {
    Image("smoothie/mango-jambo")
      .resizable()
      .aspectRatio(contentMode: .fit)
      .frame(width: 96.0)
    
    VStack {
      Text("Mango Smoothie")
      Text("Mango, Banana, Almond Milk")
      Text("\(smoothie.kilocalories) Calories")
    }
  }
}
```

## Popularity Rating
We want to add a star rating to display the smoothie's popularity. There's no built-in star view, but it's easy to drag and drop our own custom views as well once we've added them to the library.

To learn more, see the session [Add custom views and modifiers to the Xcode Library](../10649).

![][custom_drag_and_drop]

When we drop, we get a single star. However, we want 5. It's easy to repeat this by CMD+clicking the view and clicking "Repeat".

![][embed_hstack]

## Modifying Labels
To make our smoothie title "pop", we can use the inspector on the right to try various font and weight values on our view.

To clear any modifiers, we can click on the circular blue indicator next to a control. This returns the view to its inherited value. If there are no inherited values, this returns the view to its SwiftUI defaults.

![][clear_modifier]

## Previewing across different configurations
Xcode 12 allows us to duplicate a preview with a single click on the top right of the menu.

![][duplicate_preview]

Also, since previews are just views, we can inspect them too.

![][inspect_preview]

[row_view_outline]: row_view_outline.png

[drag_and_drop]: drag_and_drop.png

[live_update]: live_update.png

[live_update_variables]: live_update_variables.png

[custom_media]: custom_media.png

[large_image]: large_image.png

[add_modifier]: add_modifier.png

[custom_drag_and_drop]: custom_drag_and_drop.png

[embed_hstack]: embed_hstack.png

[clear_modifier]: clear_modifier.png

[duplicate_preview]: duplicate_preview.png

[inspect_preview]: inspect_preview.png

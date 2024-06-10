# Meet SwiftUI for spatial computing

Take a tour of the solar system with us and explore SwiftUI for visionOS! Discover how you can build an entirely new universe of apps with windows, volumes, and spaces. We’ll show you how to get started with SwiftUI on this platform as we build an astronomy app, add 3D content, and create a fully immersive experience to transport people to the stars.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10109", purpose: link, label: "Watch Video (25 min)")

   @Contributors {
      @GitHubUser(ronyfadel)
   }
}



## Introduction

* When building an app for spatial computing: the best way to build it is with SwiftUI.

* New capabiltes such as Volumes and new 3D gestures, effects and layouts only exist in SwiftUI.

* The visionOS system has been built from the ground up with SwiftUI (e.g. buttons, toggles, Home View, Control Centre)

![Overview of SwiftUI view components optimized for visionOS][ui-components]

[ui-components]: WWDC23-10109-ui-components

* SwiftUI elements adapt to the idioms of visionOS:
	* bordered buttons use a vibrant material background.
	* all buttons gain a rich hover effect that react to your eyes, hands and to pointer input.
	* buttons can automatically display a tooltip when you look at them.
	* TabView hangs by the side of your app.
	* TabView expands to display more detail just by looking at it.


## Scenes

![Kinds of Scenes on visionOS: Window, Volume, or Full Space][spatial-scenes]

[spatial-scenes]: WWDC23-10109-spatial-scenes

With spatial computing, there are 3 types of scenes that make up an app:
* **Windows**: great for building traditional and familiar interfaces (e.g. Safari, Freeform), and menus that lead into more immersive experiences (e.g. Mindfulness)

* **Volumes**: a new 3D window style: for displaying objects and experiences in a bounded space.

* **Full Spaces**: app gets complete control, hiding windows from other apps.

## Window

#### Ornaments

New concept added to SwiftUI: Ornaments.

Ornaments allow you to add accessory views relative to your app's window. They can even extend outside the window's bounds.

Create your own ornament using the `.ornament` modifier.

#### Materials

No dark or light appearance on visionOS. Materials do the hard work for you.

Secondary foreground style (`.foregroundStyle(.secondary)`) automatically uses a new vibrant treatment within its background material to increase visual weight.

### Interaction
- **Eyes**: Look at an element and use an indirect pinch gesture.
- **Hands**: reaching out and touching apps.
- **Pointer**: trackpad, hand gesture or hardware keyboard.
- **Accessibility**: e.g. VoiceOver, Switch Control.

New gestures added to SwiftUI, like `RotateGesture3D`.

![New SwiftUI gestures: RotateGesture3D, targetedToEntity, SpatialEventGesture, preferredHandAction, 3D properties on spatial gestures][spacial-swiftui-gestures]

[spacial-swiftui-gestures]: WWDC23-10109-spacial-swiftui-gestures

#### Hover effects
* Critical to making your app responsive.
* Run outside of your app's process.
* Added automatically to most controls.
* If you're using a custom control style, make sure to add hover effects to make them responsive and easy to use.

## Volumes
* To add a volume, specify `.windowStyle(.volumetric)` on your `WindowGroup`.

### Model3D

![Use Model3D in SwiftUI like an AsyncImage][model3d]

[model3d]: WWDC23-10109-model3d

* To display a 3D model, use the new `Model3D` API from RealityKit.

* A `Model3D` always loads asynchornously. Similar to `AsyncImageView`, a `Model3D` can display a placeholder view.

* Layouts like `ZStack` are automatically aware of the depth of your content.

### RealityView

* `RealityView` provides easy access to the full power of RealityKit Pro.

* `SpatialTapGesture` gives the full 3D location of the tap.

* the `tagetedToAnyEntity` view modifier provides context like the entity you tapped on and the location relative to that entity.

* `RealityView` attachments allow mixing custom SwiftUI views together, inline with RealityKit entities.

## Full Spaces

### ImmersiveSpace

* Add an `ImmersiveSpace` scene.

* Provide an id to the space so you can programmatically open it from your main window.

* To open the space, use the new `openImmersiveSpace` environment action.

### Immersion styles

![Styles of Immersion on visionOS: Mixed, Progressive, or Full][immersion-styles]

[immersion-styles]: WWDC23-10109-immersion-styles

* Mixed, Progressive and Full immersion are supported.

* With Progressive immersion, you can use the digital dial to dial in how much immersion feels right to you.

* Use the `immersionStyle` view modifier.

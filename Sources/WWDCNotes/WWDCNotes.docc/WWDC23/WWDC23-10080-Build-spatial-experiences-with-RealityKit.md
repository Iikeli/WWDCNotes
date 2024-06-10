# Build spatial experiences with RealityKit

Discover how RealityKit can bring your apps into a new dimension. Get started with RealityKit entities, components, and systems, and learn how you can add 3D models and effects to your app on visionOS. We’ll also take you through the RealityView API and demonstrate how to add 3D objects to windows, volumes, and spaces to make your apps more immersive. And we’ll explore combining RealityKit with spatial input, animation, and spatial audio.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10080", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(stevenpaulhoward)
   }
}



Speakers: John Calsbeek, RealityKit Engineer

> _RealityKit is a framework for realistically rendering, animating, and simulating 3D models and effects._

![Schematic of features included in RealityKit.][realitykit-features]

[realitykit-features]: realitykit-features.png

## How RealityKit and SwiftUI work together [02:49](https://developer.apple.com/videos/play/wwdc2023/10080/?time=169)

**SwiftUI** is how you define your views and windows.

**RealityKit** is how you add 3D elements.

Adding 3D content to a 2D window is easy using the `Model3D` view in RealityKit. [Example here](https://developer.apple.com/videos/play/wwdc2023/10080/?time=209).

The `Model3D` structure takes a content closure to include code that customizes your 3D model, and a placeholder view builder to specify what is displayed while the model is loading.

![Structure of Model3D.][model3d-struc]

[model3d-struc]: model3d-struc.png

You can create a volumetric window by creating window groups. [05:48](https://developer.apple.com/videos/play/wwdc2023/10080/?time=348)
- _Window groups act as a template that an app can use to open new windows_.

Model3D and RealityView are different. 
- **Model3D** renders a 3D model inside your window.
- **RealityView** is a new SwiftUI view for 3D models and effects, and grants more control over the scene.

Learn more about the nuances when building Immersive Spaces and what's new in SwiftUI for spatial computing through the following sessions:
- [Meet SwiftUI for Spatial Computing](https://developer.apple.com/videos/play/wwdc2023/10109/)
- [Take SwiftUI to the next dimension](https://developer.apple.com/videos/play/wwdc2023/10113)
- [Go Beyond the Window with SwiftUI](https://developer.apple.com/videos/play/wwdc2023/10111)

## What is a RealityKit entity? What is a RealityKit Component? [09:36](https://developer.apple.com/videos/play/wwdc2023/10080/?time=576)

> _An entity is a container object. If you create an empty entity from code, it won't do anything. To make an entity render or give a behavior, it must have components. Each component enables some specific behavior for an entity_.

**Entities** are containers.
**Components** enable behaviors for entities.

![Schematic of components included in RealityKit.][realitykit-components]

[realitykit-components]: realitykit-components.png

### Simple entity example

In the following example, the `model` component renders a 3D model, while the `transform` component places the entity in 3D space.

![Example of the structure of an entity.][simple-entity]

[simple-entity]: simple-entity.png

### Robust entity example

> _Every entity has a transform, but not every entity has a model. Sometimes, an entity is assembled out of multiple child entities, each with their own set of components_.

In the following example, the entity is assembled out of mulitple child entities. This gives you more power, such as being able to play individual animations on the transforms of child entities. 

![Example of the structure of an entity with nested children entities.][robust-entity]

[robust-entity]: robust-entity.png

### Gotchas

_Note: Entities need to be added to a RealityView in order to be rendered, animated, and simulated_.

_Note: XYZ conventions in RealityKit are different than XYZ conventions in SwiftUI_

![XYZ conventions in RealityKit are different than XYZ conventions in SwiftUI.][coordinate-conventions]

[coordinate-conventions]: coordinate-conventions.png

## Use RealityView to place entities in-app [12:18](https://developer.apple.com/videos/play/wwdc2023/10080/?time=738)

> _RealityView is a SwiftUI view that contains any number of entities_.

**RealityView provides:**

- **content instance** which lets you add entities to the view; this closure is asynchronous.

- **update closure** which lets you connect observable state to component properties.

- **coordinate conversion functions** to assist in the conversion of values if your app needs to dynamically adjust values when transitioning between views and entities. [More on RealityView's content instance `convert` function](https://developer.apple.com/videos/play/wwdc2023/10080/?time=865) 

- **mechanism to subscribe to events** published by entities and components.

- **attachment of SwiftUI views to entities** making it easy to position views in 3D space.

Learn more in [**Enhance your Spatial Computing App with RealityKit**](https://developer.apple.com/videos/play/wwdc2023/10081)

## Input, Animations, and Spatial Audio [15:36](https://developer.apple.com/videos/play/wwdc2023/10080/?time=936)

In order to add a gesture to RealityView, an entity must have both an **input target** component and a **collision** component.

![RealityView gestures only respond to entities with input target and collision components.][realityview-gestures]

[realityview-gestures]: realityview-gestures.png

> _It's important for the collision shape to be a reasonable approximation of the visual model. The closer the match, the more intuitive interactions with the model will be._

![Collision shapes should match the shape of your model for inutitive interactions.][collision-shapes]

[collision-shapes]: collision-shapes.png

You can change the shape of the collider inside Reality Composer Pro under the Collider component.

_Tip: you can see the collider wireframe by going to `Viewport > Collision Shapes` inside Xcode_

### Adding `DragGesture()` to an entity [18:30](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1110)

- `.targetedToEntity` modifier allows the gesture to target a specific entity
- The gesture is in SwiftUI coordinate space, so you have to convert it to RealityKit's coordinate space in order to change the entity's position

Hover effects provided by SwiftUI and RealityKit are the only way to make your app react to where you're looking. Enable a hover effect with `HoverEffectComponent`.

### Animating entities [19:53](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1193)

Animation types:
- From-to-by
- Orbit
- Time Sampled

_How to Setup an Orbit Animation_ [20:19](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1219)

### Spatial Audio in RealityKit [21:00](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1260)

Spatial Audio types:
- Spatial
    - use Directivity to emit sounds in all directions
    - or, project sounds in a specific direction
- Ambient
    - great for multi-channel files that capture sounds of an environment
    - no additional reverb is added to ambient sources; each channel of the ambience is played from a fixed direction
- Channel
    - sends the audio file channels directly to the speakers without any spatial effects

_How to add audio to your scene_ [22:01](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1321)

## Custom Systems [23:04](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1384)

### Custom Components [23:18](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1398)
> _A component contains the data controlling one aspect of a 3D experience. Components are grouped into entities. Without components, an entity does nothing. Each component supplies a single element of an entity's implementation._

In addition to using the components RealityKit provides, you can define your own components to add custom functionality.

_Example of a custom component_ [23:46](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1426)

### Custom Systems [24:32](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1472)

> _Systems contain code that act on entities and components. Taken together, entities, components, and systems, or ECS, are a tool for modeling the appearance and behavior of your 3D experience. Systems are a way to structure the code that implement your app's behaviors._

![Schematic of systems.][systems]

[systems]: systems.png

- Once a system is registered, it automatically applies everywhere in your app that you use RealityKit.

_Example of a system implementation_ [25:02](https://developer.apple.com/videos/play/wwdc2023/10080/?time=1502)

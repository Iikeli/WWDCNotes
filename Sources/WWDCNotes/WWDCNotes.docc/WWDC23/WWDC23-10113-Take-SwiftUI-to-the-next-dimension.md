# Take SwiftUI to the next dimension

Get ready to add depth and dimension to your visionOS apps. Find out how to bring three-dimensional objects to your app using volumes, get to know the Model 3D API, and learn how to position and animate content. We’ll also show you how to use UI attachments in RealityView and support gestures in your content.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10113", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(coughski)
   }
}



## Introduction

There are 3 ways to display content in visionOS:
- **Window**: 2D content
- **Volume**: bounded 3D experiences
- **Full Space**: display of unbounded 3D content

This talk focuses on **Volumes**.

## Volumes

- Can use without any windows
- Provides a fixed scale container
- Unlike windows, maintains the same size at any distance
- Horizontally aligned, and support viewing from any angle
- Doesn't take over the entire space
- Use `.windowStyle(.volumetric)` modifier on a WindowGroup

## 3D views and layout

#### Model3D

- A RealityKit view to load simple 3D scenes asynchronously
- Similar to `AsyncImage`, you can use placeholders during loading
- Use `.resizable` modifier with model, similar to `Image`
- Default alignment is against back plane. Otherwise, use `.frame(depth:alignment:)` with `.front` alignment
- Can use `.glassBackgroundEffect()` to make text labels readable in visionOS

## RealityView

Use for more involved 3D scenes

Can use RealityKit's Entity-Component system for advanced rendering

#### RealityView attachments

- Pair tagged (`.tag`) SwiftUI views with Entity inside RealityView
- Useful for adding annotations or editing affordances
- These live views can respond to state changes, run animations, and handle gestures

## 3D gestures

- Existing gestures supported in 3D
- Hands, eyes, trackpad mechanics supported

Add `InputTargetComponent` to RealityKit Entity to receive input

`CollisionComponent` is used to define shape of entity's interactive region

Can use `SpatialTapGesture` to allow user to interact with entity

`.targetedToEntity` modifier ties gestures to specific entities

Convert from SwiftUI local space to scene's coordinate space

Instead of loading another model using RealityKit, you can add a `Model3D` as an attachment

`RotateGesture3D` is another new gesture you can use

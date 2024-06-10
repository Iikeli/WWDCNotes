# Develop your first immersive app

Find out how you can build immersive apps for visionOS using Xcode and Reality Composer Pro. We’ll show you how to get started with a new visionOS project, use Xcode Previews for your SwiftUI development, and take advantage of RealityKit and RealityView to render 3D content. 

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10203", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(stevenpaulhoward)
   }
}



Speakers: Peter Zion, RealityKit Tools

## Create a new app project in Xcode [_01:06_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=66)

Two new options are presented when creating a new project for the visionOS platform:

* **Initial Scene** which allows us to specify the type of the initial scene that's automatically included in the app
    * _Note: you can always add additional scenes later_
        * There are 2 initial scenes to choose from: **Window** and **Volume**
        * **Window** primarily used for present 2D content and can be resized in their planar dimension
              * Learn more about windows in [Meet SwiftUI for Spatial Computing](https://developer.apple.com/videos/play/wwdc2023/10109)
        * **Volumes** are designed to present 3D content and cannot be adjusted by the person using the app
              * Learn more about volumes in [Take SwiftUI to the next dimension](https://developer.apple.com/videos/play/wwdc2023/10113/)
          
* **Immersive Space** gives us the opportunity to add a starting point for immersive content to our app
    * Immersive Space can be used to present unbounded content anywere on the infinite canvas
    * When your app activates this scene type, it moves from **Shared Space** to **Full Space**
    * Full Space provides access to dedicated rendering resources and can request permission to enable ARKit permissions
    * 3 styles of immersive space: **Mixed Immersion**, **Progressive Immersion**, **Full Immersion**
          * **Mixed Immersion** lets you place unbounded virtual content in a Full Space while maintaining passthrough
          * **Progressive Immersion** opens a portal for a roughly 180-degree view of your content
          * **Full Immersion** hides passthrough entirely and surrounds eople with your app's environment
    * Learn more about Immersive Spaces in [Go Beyond the Window with SwiftUI](https://developer.apple.com/videos/play/wwdc2023/10111/)
 
>  In general, we recommend apps always start in a window on this platform, and provide clear entry and exit controls so that people can decide when to be more immersed in your content.

### ContentView [_06:15_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=375)

Most of the prepared code is inside the `ContentView.swift` file, which uses new platform-specific features.

### RealityView [_07:23_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=443)

RealityView allows you to place Reality content into a SwiftUI view hierarchy and the initializer takes 2 closures as parameters: **make closure**, and an **update closure**.

* **Make closure** adds the initial RealityKit content to the view
* **Update closure** is optional but called when SwiftUI state changes
    * _Note: update closure is not a rendering update loop and is not called on every frame; only when the SwiftUI state changes_

Learn more about RealityView and gestures in [Build Spatial Experiences with RealityKit](https://developer.apple.com/videos/play/wwdc2023/10080/#:~:text=With%20RealityKit%2C%20you%20can%20augment,offers%20a%20lot%20of%20features.)

## Introduction to the Simulator [_08:56_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=535)

Mimicks what somebody would see when wearing the device. This section walks through simulator controls.

## How to use Xcode Previews for Quick Iteration [_12:10_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=730)

As with the simulator, Xcode previews are presented as a simulated device view. This section walks through the Xcode Previews controls, and mentions that there are advanced features such as creating custom camera angles. More information on this can be found in the developer documentation.

## Introduction to Reality Composer Pro [_13:33_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=813)

RealityKit content packages are Swift packages containing RealityKit content, and are processed at build time to optimize your content for runtime use. You can preview scenes by clicking on the content packages in your file hierarchy within Xcode, and can open those scenes to edit them inside Reality Composer Pro by clicking the button `Open in Reality Composer Pro` in the top right of the scene preview window.

Reality Composer Pro organizes its content into **scenes**. To create 3D content for a different scene in your app, simply create a new scene inside Reality Composer Pro that will target the new scene in your app (i.e. `immersiveScene` inside Reality Composer Pro could be used to render content inside your `appImmersiveScene` which would be different than content rendered in your app window or volume).

### 2 more key details to understand about Immersive Space [_16:00_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=959)

* Uses the inferred position of your feet as the origin of the content
* When your apps run in a full space, they can request access to additional data such as the exact position and orientation of your hands
    * _Note: this is not available in shared space_

Learn more about accessible user data in [Meet ARKit for Spatial Computing](https://developer.apple.com/videos/play/wwdc2023/10082/#:~:text=ARKit%20uses%20sophisticated%20computer%20vision,the%20palm%20of%20your%20hand.)

**Navigating Reality Composer Pro [_16:58_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=1017)**

### Create an Immersive Scene [_20:17_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=1216)

The first scene in the body property within your `App.swift` file is the one that will be presented by the app when it is launched. Add additional scenes to your app by adding them after the first scene.

* **Adding multiple scenes to your App.swift file [_21:18_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=1277)**
* **How to preview immserive scenes in Xcode Previews [22:43](https://developer.apple.com/videos/play/wwdc2023/10203/?time=1363)**
> By default, previews are clipped to default scene bounds. If it's presenting a view that loads content outside of these bounds, the content will not be visible. In order to support previewing immersive content that extends beyond these bounds, simply modify the view being prepared with .previewLayout(.sizeThatFits).
```
#Preview {
    ImmersiveView()
        .previewLayout(.sizeThatFits)
}
```
* **How to have your app open in Immserive Space [_23:48_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=1427)**

### Target SwiftUI Gestures to RealityKit Entities [_25:37_](https://developer.apple.com/videos/play/wwdc2023/10203/?time=1537)

**Gestures** allow SwiftUI views to respond to input events. **Entity targetting** is how input events can be targetted to specific Reality entities (e.g. tapping a specific 3D model in Immersive Space). For entity targetting to work, entities must have both a `CollisionComponent` and an `InputTargetComponent`

> Entity targeting is the glue that connects SwiftUI interactions to RealityKit content... In a more complex app, you can use entity targeting to trigger more sophisticated actions, such as presenting additional views, playing audio, or starting animations.

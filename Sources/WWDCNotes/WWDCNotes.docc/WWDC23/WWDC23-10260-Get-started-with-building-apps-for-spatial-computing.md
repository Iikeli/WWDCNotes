# Get started with building apps for spatial computing

Get ready to develop apps and games for visionOS! Discover the fundamental building blocks that make up spatial computing — windows, volumes, and spaces — and find out how you can use these elements to build engaging and immersive experiences.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10260", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(JohnBaer3)
   }
}



## ELI5 version

Windows (2D), Volumes (3D), and Spaces (amount of the user peripheral your app takes up) are the main considerations when building

RealityKit works with UIKit storyboards or SwiftUI interfaces

User privacy is protected through APIs giving the minimum required information to apps, unless they explicitly ask for additional info

Getting your apps to run in visionOS is as simple as adding it as a target!



## Summary of WWDC Session

Apps by default launch into a Shared Space, much like multiple apps on a macOS desktop. Apps can have one or more resizable windows containing 2D or 3D content, and users can reposition these as they wish.

Volumes (as in amount of space in a certain 3D object) allow for the display of 3D content in defined bounds, with SwiftUI scenes and RealityKit used to display your 3D content.

The level of immersion in an app can be controlled through opening a dedicated Full Space, where only your app's windows, volumes, and 3D objects appear. In the Full Space, you can use ARKit's APIs, including Skeletal Hand Tracking.

Your app in Full Space can use the Passthrough mode, which blends the surroundings and sound continuously into the person's surroundings, or the Fully Immersive space, which fills up the entire frame of view.

A variety of gestures are detected by the system and delivered as touch events, such as taps, long presses, and drags. In addition, interactions with RealityKit entities, Skeletal hand tracking, and inputs from wireless devices are recognized as input as well.

Collaborations are enabled through SharePlay and the Group Activities framework, and any window can be shared between users.

In terms of privacy, apps do not directly access data from the sensors. Instead, the system delivers events, visual cues, touch events to your app. If more sensitive data is needed, users are asked for their permission.

Xcode is used for app development, offering project management, visual UI editors, debugging tools, a simulator, and more. It also includes a SwiftUI preview provider and a 3D extension to visualize RealityKit code for a scene. An object mode is also available for quick previews of 3D layouts.

The simulator lets users move and look around in a scene using a keyboard, mouse, or compatible game controller, and interact with the app using simulated system gestures.

The Reality Composer Trace template was also added to Instruments, to find frame bottle-necks, other performance impacts.

Reality Composer Pro was introduced, a new developer tool that allows for the preview and preparation of 3D content. It offers features like particles, spatial audio preview, and more. The tool has standard materials, but also offers the open standard MaterialX to author custom materials for specific needs.

Testing and Iteration: One can send 3D scenes to their device and test content directly without having to build an app, which is advantageous for iteration times. Unity also offers the ability to write apps for spatial computing without needing any plugins.

Building New Apps: There are two ways to get started with building apps for spatial computing - designing a new app from scratch or converting an existing app to function in the spatial computing platform. For new apps, there are two app template options: Initial Scene Type (either 'Window' or 'Volume') and Immersive Scene Type ('Space'). SwiftUI is used to generate a working app that integrates familiar buttons with 3D objects.

Converting Existing Apps: Existing iPhone and iPad apps can be easily brought into the spatial computing platform. The apps maintain their original look and feel but also adopt native spacing, sizing, and re-layout features of the platform.

The platform provides gesture recognizers for 3D interactions, and attachments can be used to position SwiftUI elements inside the 3D scene.

They're an extension of a Window, ideal for 3D content, can host multiple views, and are built for Shared Space.

From 20:00, there is an walkthrough of a Hello World app in Xcode. It showcases how different elements such as text, images, and buttons can be navigated using tap gestures, and how 3D content can be incorporated alongside 2D UI in a 'window'. He also demonstrates the use of volumes and spaces, which are larger containers for 2D and 3D content, and can offer different levels of immersion depending on user preferences.

Additional Resources: For more advanced app development, watching sessions on the principles of spatial design, building apps with SwiftUI and RealityKit, and creating 3D content are recommended.

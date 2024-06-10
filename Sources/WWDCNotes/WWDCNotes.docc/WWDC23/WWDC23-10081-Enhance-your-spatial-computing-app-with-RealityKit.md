# Enhance your spatial computing app with RealityKit

Go beyond the window and learn how you can bring engaging and immersive 3D content to your apps with RealityKit. Discover how SwiftUI scenes work in tandem with RealityView and how you can embed your content into an entity hierarchy. We’ll also explore how you can blend virtual content and the real world using anchors, bring particle effects into your apps, add video content, and create more immersive experiences with portals.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10081", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(stevenpaulhoward)
   }
}



Speakers: Yujin Ariza, RealityKit Engineer

Refresher: 
- **Entities** are container objects.
- **Components** define specific behavior on entities.
- **Systems** act on both entities and components to add functionality.
- **RealityView** is an API that acts as a bridge between SwiftUI and RealityKit

## How to embed SwiftUI views into RealityKit content using attachments in RealityView [02:05](https://developer.apple.com/videos/play/wwdc2023/10081?time=125)

> Attachments are a useful way to embed SwiftUI content into your RealityKit scene.

[Example app that demonstrates how to put text labels and views alongside 3D content using attachment views](https://developer.apple.com/videos/play/wwdc2023/10081?time=147).

**2 parts to settings up attachments**
1. Add the parameter in the make closure of your RealityView
2. Include the attachments view builder to your RealityView

The **attachments view builder** is where you can provide SwiftUI views that you want to add to your RealityKit content. You can then tag those views and call them as entities **inside your make closure** using the call `entity(for:)`.
- How to use multiple attachments inside of a make closure [04:32](https://developer.apple.com/videos/play/wwdc2023/10081?time=272)

![Structure of attachments.][attachments]

[attachments]: WWDC23-10081-attachments

You can also update entities inside of the update closure whenever there are changes to SwiftUI view state.

![Attachments with update closure.][attachments-update]

[attachments-update]: WWDC23-10081-attachments-update

## How to add Video Playback in RealityKit scenes [06:17](https://developer.apple.com/videos/play/wwdc2023/10081?time=377)

`VideoPlayerComponent` is a new component type in RealityKit used for embedding video content inside a 3D scene.
- _Reminder: components define behavior that you attach to entities_.

**To play a video using `VideoPlayerComponent`**:
1. Load a video file from resources bundle
2. Create an `AVPlayer` instance
3. Create a `VideoPlayerComponent`
4. Attach component to an entity

![Video Player Component Flow.][videoplayercomponent]

[videoplayercomponent]: WWDC23-10081-videoplayercomponent

- Since RealityKit is a 3D framework, video is represented as an entity with a mesh so that you can move and position it in 3D space.
- `VideoPlayerComponent` also supports **Passthrough Tinting** by setting the `isPassthroughTintingEnabled` property to true.
- You can also subscribe to `VideoPlayerEvents` by calling the subscribe function inside RealityViews content, and specifying the event type and entity [10:39](https://developer.apple.com/videos/play/wwdc2023/10081?time=639).

Learn more about video content including 3D videos in [Deliver video content for spatial experiences](https://developer.apple.com/videos/play/wwdc2023/10071).

## How to use Portals [11:06](https://developer.apple.com/videos/play/wwdc2023/10081?time=666)

**Portals** create openings to other worlds that are visible through mesh surfaces. Entities inside of these worlds use separate lighting, and are masked by portal geometry.

To make a portal, you must first create a **World**. You doing this by adding a `World` component to an entity. Add content to the world by attaching entities as children of the world entity. 
_Note: entities in a world are only visible through a portal surface_

Once you have your World entity, you create a mesh with a portal material, then add a portal component targeted to the world entity. [Example code walkthrough here](https://developer.apple.com/videos/play/wwdc2023/10081/?time=791).

![Schematic of portals.][portals]

[portals]: WWDC23-10081-portals

- Basically there's a separate component called `World` which is what you use in portals and I think is the only way to render portals
- Lighting is unique and can learn more in another session
- Show schematics for how Portals are built
- Link example he creates

## How to use the Particle Emitters API [15:04](https://developer.apple.com/videos/play/wwdc2023/10081?time=904)

`ParticleEmitterComponent` provided in RealityKit can be created either Reality Composer Pro or via RealityKit at runtime.

> _The ParticleEmitterComponent contains many properties that control various aspects of particle look and behavior._

Code walkthrough of building a particle system [15:49](https://developer.apple.com/videos/play/wwdc2023/10081/?time=949).

## How to use Anchors in RealityKit [17:10](https://developer.apple.com/videos/play/wwdc2023/10081?time=1030)

**Anchors** in RealityKit can be used to place content on surfaces, such as walls, floors, or locations relative to head and hands.

Anchors support 2 tracking modes:
1. `.continuous`
- anchor entity moves along with the anchor over time, such as when your head moves.
2. `.once`
- anchor entity will not move after being positioned once.

You can listen to when an entity becomes anchored by subscribing to the `AnchoredStateChanged` event in RealityKit.

Anchor transforms are **not** accessible to the app. In order to get access to anchor transforms, you will need to use ARKit.

Code walkthrough of setting an anchor in your app [18:18](https://developer.apple.com/videos/play/wwdc2023/10081/?time=1098).
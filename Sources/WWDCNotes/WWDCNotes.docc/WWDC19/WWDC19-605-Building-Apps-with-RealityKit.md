# Building Apps with RealityKit

Gain a practical understanding of RealityKit capabilities by developing a game using its easy-to-learn API. Learn the recommended approach for loading assets, building a scene, applying animations, and handling game input. See how entities and components express the powerful elements of RealityKit while providing flexibility for customization. Find out how to take advantage of built-in networking and get details about extending the game into an immersive muliti-player experience.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/605", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## RealityKit recap

- It’s ARKit made it easy.
- Allows to seamlessly blend rendered content with a real world environment.

## Add interaction (Hit testing)

- Hit testing works by turning a 2D point that was tapped on your device's screen into a ray in our virtual scene. That ray is then cast into the scene and RealityKit finds all of the objects that are intersected by the ray. Any entities that were intersected by the ray are returned and you now know what objects lie under the tap.
- In order for hit testing to work, our entities must be/have a collision type shape.
- Collision shapes are geometric shapes.
- Can be generated automatically (must ask explicitly when loading the model), by using the entity visual bounds.

## Animation

- Transform: Position, Scale, Rotation
- Asset: Animations that are baked into the assets that are loaded in from USDZs or reality files.

## Occlusion Materials 

- Occlusion materials are invisible, but when applied to geometry in a scene they hide virtual content behind them revealing the video pass through. Useful to hide (part of) entities that should be visible at the current moment

## Components

- RealityKit uses the entity component design pattern to build objects within the virtual world.
- An entity itself is comprised of pieces called "components." 
- These components define specific behaviors and data that can be added to individual entities. Using entities and components allows for reuse of code and is flexible to use.
- It’s a `Struct` conforming to the `Component` protocol

## Multiplayer

- Automatic scene synchronization
- Built on `MultipeerConnectivity`
- The models must be loaded separately by the different clients.
- Entity Ownership
  - It’s the right to modify an Entity
  - By default this is given to the owner of the AR experience (the host)
  - The host can decide to give this right to others (the clients)
  - To do this, the client can ask for the ownership of an entity, and the host can grant it.
  - This can be done automatically with the `ownershipTransferMode` property
  - Once the ownership has been transferred, even the host has to request for ownership.

We can also add local only entities/components, which are components not shared with other players.This is useful for showing what belongs to the device user, and not the opponents on a game for example.

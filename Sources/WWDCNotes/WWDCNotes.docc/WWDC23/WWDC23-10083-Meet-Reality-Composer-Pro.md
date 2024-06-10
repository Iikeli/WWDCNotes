# Meet Reality Composer Pro

Discover how to easily compose, edit, and preview 3D content with Reality Composer Pro. Follow along as we explore this developer tool by setting up a new project, composing scenes, adding particle emitters and audio, and even previewing content on device.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10083", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(halmueller)
   }
}



# Chapters
[00:00 - Introduction](https://developer.apple.com/videos/play/wwdc2023/10083/?time=0)    
[01:15 - Project setup](https://developer.apple.com/videos/play/wwdc2023/10083/?time=75)    
[02:47 - UI navigation](https://developer.apple.com/videos/play/wwdc2023/10083/?time=167)    
[04:08 - Composing scenes](https://developer.apple.com/videos/play/wwdc2023/10083/?time=248)    
[07:08 - Particle emitters](https://developer.apple.com/videos/play/wwdc2023/10083/?time=428)    
[13:23 - Audio authoring](https://developer.apple.com/videos/play/wwdc2023/10083/?time=803)    
[17:39 - Statistics](https://developer.apple.com/videos/play/wwdc2023/10083/?time=1059)    
[19:26 - On-device preview](https://developer.apple.com/videos/play/wwdc2023/10083/?time=1166)    
[19:59 - Wrap-up](https://developer.apple.com/videos/play/wwdc2023/10083/?time=1199)    

Session is a walkthrough of Reality Composer Pro, using [Diorama sample app](https://developer.apple.com/documentation/visionos/diorama).

Can launch RCP directly from Xcode->Developer Tools menu.
Or can create a RCP project by creating a new Xcode xrOS project, which creates a Reality Composer Pro Package.

# Reality Composer Pro UI navigation:
- 3D viewport (center): navigate with WASD and arrow keys, or with a paired game controller.
- Hierarchy panel (left) for object selection and reorganizaation.
- Inspector panel (right): edit properties of selected objects. "Add component" button at bottom shows available built-in objects.
- Editor panel (bottom): Project browser, Shader Graph, Audio Mixer, Statistics.

3 ways to add assets:
- Import button in project browser: import existing assets (USDZ, wav, more)
- Content library ("+" button at top right): curated assets (USDZ, materials, more) from Apple.
- Object Capture. See ["Meet Object Capture for iOS" session](https://developer.apple.com/videos/play/wwdc2023/10191/).

[05:03: Walkthrough of building diorama from imported assets, plus library assets.](https://developer.apple.com/videos/play/wwdc2023/10083/?time=303)

Imported assets can be replaced with new version, e.g. change the style of _all_ location pins by updating one file.

# Particle emitters

Demo: Clouds composition. Add article emitters (freestanding asset, or attach to an existing asset).

Build a Cloud Chunk. Particle Emitter 
tinkering, starting with the "Impact" particle emitter preset.
Change 3D viewport background color. Live playback (top of inspector panel). Large number of particles slows performance.
Check "isLocalSpace" so that parent's translation/rotation/scaling will also apply to the emitter.

New scene, Cloud A. Add Cloud Chunk multiple times, positioned for realism.

Back to diorama. Add group Clouds, which has Cloud A, Cloud B, Cloud C. Preview just the "Clouds" group with Playback.

# Audio authoring

Audio files. Can be played on one or more objects. An object can play one or multiple audio files.

Audio sources: spatial (comes from a particular object), ambient (e.g. wind from the east, no matter how far east you travel), channel (background music).

Example: animated bird (USDZ), with two bird call audio files attached as Spatial audio source. Audio File Group randomly selects one of its members to play back.
Preview: playback of animation and audio of all birds and calls in scene. Additional work needed to control from Swift; see [Work with Reality Composer Pro content in 
Xcode](https://developer.apple.com/videos/play/wwdc2023/10202/).

# Statistics

For performance optimization.

Note that diorama base has many more triangles than the terrain itself. Replace base asset with much simpler version. Reduces triangle count by over half.

# On-device preview

Drop-down at upper right, select actual device. Object appears in AR, can pinch, drag, zoom the scene.

# Wrap-up
- [Explore materials in Reality Composer Pro (Shader Graph details)](https://developer.apple.com/videos/play/wwdc2023/10202/)
- [Work with Reality Composer Pro content in Xcode (make scene interactive)](https://developer.apple.com/videos/play/wwdc2023/10273/)

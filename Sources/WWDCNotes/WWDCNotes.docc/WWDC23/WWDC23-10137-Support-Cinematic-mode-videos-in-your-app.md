# Support Cinematic mode videos in your app

Discover how the Cinematic Camera API helps your app work with Cinematic mode videos captured in the Camera app. We’ll share the fundamentals — including Decision layers — that make up Cinematic mode video, show you how to access and update Decisions in your app, and help you save and load those changes.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10137", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(ronyfadel)
   }
}



## What is cinematic mode?

* a "tiny film crew right in your pocket".
* brings you a camera with shallow depth of field and natural focus falloff.
* a director, who directs the attention and narrative by changing the focus.
* a focus puller who anticipates keyframes ahead of time and creates smooth transitions between focus points.
* It's captured in the Camera app on iPhone 13 and newer.

## Cinematic API
Available on macOS 14, iOS 17, iPadOS 17 and tvOS 17.

### Dataflow
* Two assets: Rendered asset and Cinematic mode asset
* Rendered asset can be shared and played as a regular Quicktime movie.
* Cinematic mode asset: has all the information needed to create the Rendered asset. Allow non-destructive post-capture edits.

#### Cinematic mode asset
* has multiple tracks
	* the original QuickTime movie: can be HDR/SDR, 1080p @ 30 fps on iPhone 13 and up to 4k @ 24/25/30 fps on iPhone 14.
	* Disparity used for focus and rendering the shallow depth of field.
	* Metadata:
		* rendering attributes which hold focus disparity (chosen by the cinematic engine) and aperture as an f number (chosen by the user)
		* Cinematic script which holds all the automatic scene detections, and the focus decisions deciding which detection to keep in focus.

### Playback
* To pick a Cinematic mode asset, use `PhotosPicker` and filter for `.cinematicVideos`.
* Set the request options to get the `.original` version.
* Can be played back as a regular movie using AVPlayer and AVPlayerItem.
* To exploit to the max, we'll need to add a custom video compositor.
* Create a session using`CNRenderingSession`, and an `AVMutableComposition`.
* In the custom compositor you can set the f number by setting a `fNumber` on the `frameAttributes`.

### Editing
* each cinematic frame holds all detections for a given point in time.
* Base focus decisions are decided by the system. They can be overriden by a Weak user decision (overriden by the following Base decision), or a Strong user decision that will keep focus on a subject until the next user decision.
* to draw detection boxes, you can get the current script frame using `cinematicScript.frame(at: scriptFrameTime, tolerance: tolerance)`.
* To add a new decision, create a `CNDecision` and add it to the `cinematicScript`.
* The `cinematicScript` changes have a `dataRepresentation` property that can be written to disk.

# What's new in PencilKit

PencilKit helps power creativity, writing, drawing, and animation in your iPad apps. Explore the latest improvements to our drawing and annotation framework, and discover how you can take advantage of APIs like PKToolPicker, PKCanvasView, and PKStroke to support new features in illustration and writing apps. 

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10107", purpose: link, label: "Watch Video (10 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## New fatures

- Rich-selection experience:
  - Double-tap to select a word
  - Double-tap again to select a line
  - Grab selection handles to expand your selection

- Enhanced tap-and-pan gesture: you can make noncontiguous selections by brushing over specific strokes
- Add space between scribbles/notes:
  - Tap on the space between two lines
  - Tap `Insert Space` from the callout bar
  - Use the grab handle to adjust the amount of space

- New color picker:
  - Saved colors
  - Eye dropper

- Catalyst support

## Pencil vs hand drawing

- New (iOS 14 only) system settings for users to choose if they would like to draw only with the pencil or with fingers as well.
- All apps need to respect that, if you have your own drawing engine, check [`UIPencilInteraction.prefersPencilOnlyDrawing`][prefersPencilOnlyDrawing]
- This can also be toggled directly on the tool picker:
![][prefersImage]

- [`PKCanvasView`][PKCanvasView] has been updated to reflect this preference with the new [`drawingPolicy`][drawingPolicy], these are the possible values/behaviors:
  - `anyInput`: allows drawing on the canvas from any input source
  - `pencilOnly`: pencil touches are the only input that draw on the canvas
  - `default`:
    - if the tool picker is displayed, it follows `UIPencilInteraction.prefersPencilOnlyDrawing`
    - if the tool picker is hidden, it's the same as `pencilOnly`

- If you'd like to hide the `Draw with Finger` option in the tool picker (for example when your app is a pencil-only app) set the [`PKToolPicker.showsDrawingPolicyControls`][showsDrawingPolicyControls] to `false`.

- You can also create your own `PKToolPicker` instances:
  - Need to be retained by your app
  - Each toolpicker can be used with different states with different canvases

## Strokes Access

- PencilKit in iOS 14 provides access to strokes, for more, check session [`Inspect, modify, and construct PencilKit drawings`][20-10148]

[20-10148]: ../10148
[showsDrawingPolicyControls]: https://developer.apple.com/documentation/pencilkit/pktoolpicker/3552394-showsdrawingpolicycontrols
[PKCanvasView]: https://developer.apple.com/documentation/pencilkit/pkcanvasview
[drawingPolicy]: https://developer.apple.com/documentation/pencilkit/pkcanvasview/3552388-drawingpolicy
[prefersPencilOnlyDrawing]: https://developer.apple.com/documentation/uikit/uipencilinteraction/3552414-preferspencilonlydrawing

[prefersImage]: WWDC20-10107-prefers
# Inspect, modify, and construct PencilKit drawings

Make Apple Pencil an even more useful tool for drawing and writing within your app. With PencilKit, you can delve into the strokes, inks, paths, and points that comprise a drawing, use these to build features that use recognition, and modify drawings in response to input. Discover how you can dynamically generate shapes and drawings and learn more about APIs like PKDrawings and PKStrokes.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10148", purpose: link, label: "Watch Video (16 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- In iOS 14, we have access to PencilKit's data model, this enable us to:
  - Inspect the contents of what the users drew
  - React to what was drawn
  - Manipulate existing drawings
  - Dynamically create new drawings from scratch

## PKStroke

![][strokeImage]

A [`PKStroke`][PKStroke] is composed by:

- A path, which provides the shape of the stroke
- An ink, which describes the appearance of the stroke, its color and type
- A transform, giving us the orientation and position of the stroke
- A mask, created when the pixel eraser is used to erase only a portion of a stroke
- The renderBounds, a.k.a. the bounding box that encompasses the entirety of the stroke when it is rendered, the renderBounds accounts for the effect of all the stroke properties including the path, ink, transform, and mask. 

## PKStrokePath

- Describes the shape of the stroke, and the appearance of that shape as it changes along the path.
- Gives you the width of the stroke at any point
- The stroke path is a uniform cubic, B-spline of PencilKit stroke points
   - The contents of a path the control points for the B-Spline
![][splineImage]

-  
   - To get points on the actual path, we need to interpolate the spline using [`interpolatedPoints(strideBy:)`][interpolatedPoints(strideBy:)].
![][interpolationImage]

```swift
for point in path.interpolatedPoints(strideBy: .distance(50)) {
  draw(point)
}
```

## PKStrokePoint

![][pointImage]

A `PKStrokePoint` describe how a stroke appears at a certain location. It is composed by:

- A location
- A size, for marker strokes won't be square
- A rotation angle, or azimuth
- An opacity
- A force (same value from `UITouch` when the stroke was drawn)
- A altitude (same value from `UITouch` when the stroke was drawn)
- A time offset from path creation date

[PKStroke]: https://developer.apple.com/documentation/pencilkit/pkstroke
[PKStrokePath]: https://developer.apple.com/documentation/pencilkit/pkstrokepath
[PKStrokePoint]: https://developer.apple.com/documentation/pencilkit/pkstrokepoint
[interpolatedPoints(strideBy:)]: https://developer.apple.com/documentation/pencilkit/pkstrokepath/3595222-interpolatedpoints

[strokeImage]: ../../../images/notes/wwdc20/10108/stroke.png
[splineImage]: ../../../images/notes/wwdc20/10108/spline.png
[interpolationImage]: ../../../images/notes/wwdc20/10108/interpolation.png
[pointImage]: ../../../images/notes/wwdc20/10108/point.png
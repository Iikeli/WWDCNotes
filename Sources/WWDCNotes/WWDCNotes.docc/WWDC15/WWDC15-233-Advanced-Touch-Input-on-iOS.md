# Advanced Touch Input on iOS

Learn about the touch input and drawing pipelines. Gain specific insights in how best to design your app to minimize latency in receiving touches and maximizing the performance of drawing content on the screen. Explore new API in UIKit and learn best practices for faster and smoother input.

@Metadata {
   @TitleHeading("WWDC15")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc15/233", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Low Latency Core Animation

- The Core animation work starts as soon as the app commits, no need to wait for the next frame to come first
- Done automatically from iOS 9, no extra logic necessary
- Disabled for animations (`CAAnimation`s and `UIView`'s animations)
- Works also with [`CAEAGLLayer`][CAEAGLLayer] and [`CAMetalLayer`][CAMetalLayer], in this case you can use [`presentsWithTransaction`][presentsWithTransaction] to make sure your metal layer is in sync with your Core Animation layer (otherwise it might happen that the metal layer renders before)

## Touch Coalescing

From the iPad Air 2, iOS devices ship with a 120-hertz touch scan update rate, this means that devices get twice as much touch events for example when dragging/drawing.

iOS will still deliver new touch events at the beginning of a new frame, however you will have access to possibly two touch events instead of just one.

To get those touches, there's a new API: `UIEvent`'s [`coalescedTouches(for:)`][coalescedTouches(for:)], which returns an array of all the coalesced touches since the last time iOS delivered that touch to your app.

Usage example, from:

```swift
for touch in touches {
  let line = lineForTouch(touch)
  addTouchSample(touch, toLine: line)
}
```

to:

```swift
for touch in touches {
  let line = lineForTouch(touch)
  for coalescedTouch in event.coalescedTouchesForTouch(touch) {
    addTouchSample(coalescedTouch, toLine: line)
  }
}
```

## Touch Prediction

Use this API to get even lower latency in your apps.

How it works:

- iOS looks at the touches that are delivered to your app, and uses a set of highly tuned algorithms to determine where the user's finger looks like it's going at this time
- And as we get new touch samples, we will update our prediction and deliver new predicted touches to your app

New `UIEvent` API: [`predictedTouches(for:)`][predictedTouches(for:)]

Usage example:

```swift
for touch in touches {
  // Like before
  let line = lineForTouch(touch)
  for coalescedTouch in event.coalescedTouchesForTouch(touch) {
    addTouchSample(coalescedTouch, toLine: line)
  }

  // Remove old predicted touches
  removePredictedSamplesFromLine(line)

  // Add new predicted touches
  for predictedTouch in event.predictedTouchesForTouch(touch) {
    addPredictedTouchSample(predictedTouch, toLine: line)
  }
}
```

[CAEAGLLayer]: https://developer.apple.com/documentation/quartzcore/caeagllayer
[CAMetalLayer]: https://developer.apple.com/documentation/quartzcore/cametallayer
[presentsWithTransaction]: https://developer.apple.com/documentation/quartzcore/cametallayer/1478157-presentswithtransaction
[coalescedTouches(for:)]: https://developer.apple.com/documentation/uikit/uievent/1613808-coalescedtouches
[predictedTouches(for:)]: https://developer.apple.com/documentation/uikit/uievent/1613814-predictedtouches
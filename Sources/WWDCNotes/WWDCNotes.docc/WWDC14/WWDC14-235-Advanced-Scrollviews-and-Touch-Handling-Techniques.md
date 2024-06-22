# Advanced Scrollviews and Touch Handling Techniques

Scrollviews build on gesture recognizers and underlying multi-touch mechanics to provide a fundamental piece of the iOS user experience. Gain a broader understanding of the iOS touch handling architecture through practical real-world examples. Discover advanced tips and tricks for combining scrolling with other touch handling techniques to create delightful user interfaces.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/235", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Code sample](https://github.com/suvov/TouchDemo)

In this session we mock how we can pull Spotlight by drag a finger down on the home screen.

## Transparent overlays

The idea is to:

1. add a scroll view with our drawer content (like spotlight) on top of our main view controller
2. make the scroll view `contentSize` equal to the drawer content + the main view controller size
3. set the scroll view offset to the height of the drawer, so that it's completely hidden at start

If we build the app after these three steps, we can swipe down to display the drawer, however we cannot interact with the main view controller, as all touches are detected by the transparent scroll view.

One way to solve this is to:

1. set `userInteractionEnabled = false` on the scroll view
2. add the scroll view pan gesture to the main view: `self.view.addGestureRecognizer(myScrollView.panGestureRecognizer)`

However this is not enough, because we can now interact with the main view controller and can swipe down to show our drawer, but we cannot interact with the drawer, as all the touches (beside swipes) will be ignored by the scroll view and hit the main view controller instead.

[`hitTest(_:with:)`][hitTest(_:with:)] is the method that's used when a new touch comes down on screen to figure out where we should deliver the touch to, and what gesture recognizers should end up being involved in looking at that touch.

This is the pseudo code of the default `hitTest(_:with:)` implementation:

```swift
func hitTest(point: CGPoint, withEvent event: UIEvent) -> UIView? {
  if point(inside: point, with: event) {
    for /* each subview, in reverse order */ {
        if let hitView = /* recursive call on subview */ {
          return hitView
        }
      }
      return self
    }
  return nil
}
```

Because of this behavior, by default, `UIScrollView` returns self even when the touch hits no content within the scroll view, therefore the correct way to fix our transparent overlay is to subclass `UIScrollView` and override `hitTest(_:with:)`:

```swift
// this is implemented in our scroll view subclass, e.g., OverlayScrollView.
override func hitTest(point: CGPoint, withEvent event: UIEvent) -> UIView? {
    // get the default response from UIScrollView.
  let hitView = super.hitTest(point, withEvent: event)

  // if we don't hit anything from our scroll view content, return nil
  if hitView == self {
    return nil
    }

  return hitView
}
```

Once we replace the original scroll view with our subclass, we need to remove `userInteractionEnabled = false` on the scroll view. 

> Note that we still must keep the `self.view.addGestureRecognizer(myScrollView.panGestureRecognizer)`. This is because we want pan gestures to be still forwarded to the scroll view, as the new scroll view hit test won't catch pan gestures happening in the transparent area.

## Dragging while scrolling

In this second part of the session the team implements dragging and dropping views from the drawer to the main view and viceversa. The way this is done is by adding a long press to each relevant view, and then handle the gesture as shown here:

```swift
/* for each view */
view.addSubview(dot) // view can be the drawer or the container view

let longPress = UILongPressGestureRecognizer(target: self, action: #selector(ViewController.handleLongPress(_:)))
// 👇🏻 UIGestureRecognizers cancel touches in their view once they've recognized, setting this to false avoids that.
longPress.cancelsTouchesInView = false
dot.addGestureRecognizer(longPress)
/* for each view */

/* long press handling */
// note that this handles gesture coming from both the drawer and the main container view.
func handleLongPress(_ gesture: UILongPressGestureRecognizer) {
  if let dot = gesture.view {
    switch gesture.state {
    case .began:
      grabDot(dot, withGesture: gesture)
    case .changed:
      moveDot(dot, withGesture: gesture)
    case .ended, .cancelled:
      dropDot(dot, withGesture: gesture)
    default:
      print("gesture state not implemented")
    }
  }
}

func grabDot(_ dot: UIView, withGesture gesture: UIGestureRecognizer) {
    //  any time we re-parent a view, we need to watch out for the possibility that the origin 
    // of the new view is not the same as the origin of the old view.
  dot.center = view.convert(dot.center, from: dot.superview)
  
  // 👇🏻 we move the dot from the drawer/canvasView to the main view.
  // This also assures that the view is on top of all other views.
  view.addSubview(dot)
  
  UIView.animate(withDuration: 0.2, animations: { () -> Void in
    // we make the view slightly bigger
    dot.transform = CGAffineTransform(scaleX: 1.2, y: 1.2)
    // we make the view slightly transparent
    dot.alpha = 0.8
    // we center the view to the touch point
    self.moveDot(dot, withGesture: gesture)
  })
}

func moveDot(_ dot: UIView, withGesture gesture: UIGestureRecognizer) {
  dot.center = gesture.location(in: view)
}

func dropDot(_ dot: UIView, withGesture gesture: UIGestureRecognizer) {
    // we make the view return to its natural opacity and size.
  UIView.animate(withDuration: 0.2, animations: { () -> Void in
    dot.transform = CGAffineTransform.identity
    dot.alpha = 1.0
  })
  
  // Depending on where the gesture ended, here we move the view to 
  // either the drawer or the canvas.
  let locationInDrawer = gesture.location(in: drawerView)
  if drawerView.bounds.contains(locationInDrawer) {
    drawerView.contentView.addSubview(dot)
  } else {
    canvasView.addSubview(dot)
  }
  // like we did on grabDot, we need to make sure that the view stays in the expected 
  // position when re-parenting.
  dot.center = view.convert(dot.center, to: dot.superview)
}
```

This works great, however we can't swipe up/down the drawer while we have one (or more) UILongPressGestureRecognizer happening. We want to be able to simultaneously be dragging one of these views around and scrolling the ScrollView with another finger.

The challenge here is that, by default:

- for long press recognizers, we can interact with multiple views at the same time because those views are siblings, so their gesture recognizers don't interact with one another
- the dots are subviews of the view that has the pan gesture recognizers, so the long press gestures and the pan gesture recognizer do interact, and, by default the behavior of gesture recognizers that interact is to be mutually exclusive

We can change this by becoming `UIGestureRecognizerDelegate` for the long press gestures and implement the following delegate method:

```swift
func gestureRecognizer(
    _ gestureRecognizer: UIGestureRecognizer, 
    shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer
) -> Bool {
    // In your own applications you should be much more specific here because 
    // it would be an easy source of bugs to just return true to any gesture 
    // recognizer that you're asked about.
  return true
}
```

This works, as we can now drag a view and scroll with another finger. However, there's another issue: because we return `true` in the method above, we can now use the finger used for the long press gesture also to swipe up/down the drawer.

We want to do something when the long press starts, to prevent the pan from recognizing with that touch:

- We want to allow the long press to continue so we can drag the dot
- But we just want to stop the pan (just for that touch)

The way to fix this is to disable and re-enable the pan gesture when a new long press starts:

```swift
// disable and re-enable scrollview's pan gesture recognizer so the drawer can't be opened with moving the dot view
// disabling will cause it to stop tracking all the touches which it was tracking (including the long press)
// re-enabling will allow it to be ready to track new touches that might start
scrollView.panGestureRecognizer.isEnabled = false
scrollView.panGestureRecognizer.isEnabled = true
```

## Highlighting objects

If we swipe slowly on a view in the canvas, the long press gesture recognizer might kick in before being cancelled. This will cause our view highlight logic to start before immediately going back to the original state.
If we do the same in the drawer, this won't happen, because of `UIScrollView`'s [`delaysContentTouches`][delaysContentTouches] property, which delays handling the touch-down gesture until it can determine if scrolling is the intent.

By default a `UIScrollView` comes with three gesture recognizers:

- Pan Gesture Recognizer - for pan gestures
- Pinch Gesture Recognizer - for pinch gestures
- Touch Delay Gesture Recognizer - which never recognizes and is just as a way to delay touch delivery to the views in the scroll view

This third gesture has `delaysTouchesBegan` set to `true`, which delays delivery of `touchesBegan` and the entire touch sequence until that gesture recognizer either recognizes or fails.

To make a gesture recognizer with this behavior, we need to subclass `UIGestureRecognizer`:

```swift
class TouchDelayGestureRecognizer: UIGestureRecognizer {
  var timer: Timer?
  
  override init(target: Any?, action: Selector?) {
    super.init(target: target, action: action)
    delaysTouchesBegan = true // ⚠️
  }
  
  override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent) {
    // start timer
    timer = Timer.scheduledTimer(
        timeInterval: 0.15,
      target: self, 
      selector: #selector(TouchDelayGestureRecognizer.fail),
      userInfo: nil, 
      repeats: false
    )
  }
  
  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent) {
    fail()
  }
  
  override func touchesCancelled(_ touches: Set<UITouch>, with event: UIEvent) {
    fail()
  }
  
  func fail() {
    // As soon as the touch ends or gets cancelled, we set the state to failed,
    // and that will allow that touch to go through and get delivered to the view.
    state = .failed
  }
  
  override func reset() {
    // clear and reset the timer
    timer?.invalidate()
    timer = nil
  }
}
```

Once we have this new definition, we need to add it to our canvas view:

```swift
// we pass no target or action, as this recognizer never fires.
let touchDelay = TouchDelayGestureRecognizer(target: self, action: nil)
canvasView.addGestureRecognizer(touchDelay)
```

## Touching small objects

If some views are very small, it might be hard for the user to interact with them. To overcome this challenge, one way is to override the [`point(inside:with:)`][point(inside:with:)] method and mock-enlarge the touch area just in that function:

```swift
override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
  var touchBounds = bounds

  // 22.0 is arbitrary, but a good rule of thumb.
  if layer.cornerRadius < 22.0 {
    let expansion = 22.0 - layer.cornerRadius
    touchBounds = touchBounds.insetBy(dx: -expansion, dy: -expansion)
  }
  return touchBounds.contains(point)
}
```

[point(inside:with:)]: https://developer.apple.com/documentation/uikit/uiview/1622533-point
[delaysContentTouches]: https://developer.apple.com/documentation/uikit/uiscrollview/1619398-delayscontenttouches
[hitTest(_:with:)]: https://developer.apple.com/documentation/uikit/uiview/1622469-hittest
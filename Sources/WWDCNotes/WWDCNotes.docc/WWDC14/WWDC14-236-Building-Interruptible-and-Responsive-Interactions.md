# Building Interruptible and Responsive Interactions

Learn how to fluidly transition interactive UI elements from gesture-driven control to animated transitions. Take advantage of new iOS 8 behavior to smoothly transition between several animations on the same view. Discover architectural approaches to interfaces which remain interactive while they animate.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/236", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Gesture to Animation

When the user flicks a view, we don't want to start a standard animation with a standard speed, instead we want to use the gesture speed:

<video autoplay muted loop>
  <source src="WWDC14-236-flick">
</video>

- `UIPanGestureRecognizer` use `func velocityInView(view: UIView) -> CGPoint` (pan velocity)
- `UIPinchGestureRecognizer` use `var velocity: CGFloat { get }` (velocity of the scale)
- `UIRotationGestureRecognizer` use `var velocity: CGFloat { get }` (angular velocity for that rotation)

How do we start our animation at that velocity?

### Option 1: `UIView.animate(withDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:)`

From iOS 7 we have [`UIView.animate(withDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:)`][animate(withDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:)], where we pass an `initialSpringVelocity`.

This velocity is a normalized velocity in a normalized coordinate space, we want to normalize it based on the total distance that our view is going to travel during the animation:

- first, we have to calculate the distance (from where we start to where we want to go) in points (e.g., 100 pt)
- then, we take our initial velocity (from the gesture recognizer) in points/sec (e.g., 50 pt/sec)
- lastly we normalize the view by dividing our initial velocity by the distance (e.g., 50 pt/sec / 100 pt = 0.5 units/sec)
- we can now pass this normalized value in `initialSpringVelocity`

### Option 2: `UIDynamicAnimator`

Setup:

```swift
var dynamicAnimator: UIDynamicAnimator?
let dynamicItemBehavior = UIDynamicItemBehavior(items: nil)

override func viewDidLoad() {
  super.viewDidLoad()

  dynamicAnimator = UIDynamicAnimator(referenceView: view)
  dynamicItemBehavior.resistance = 3.0
  dynamicItemBehavior.angularResistance = 3.0
  dynamicAnimator!.addBehavior(dynamicItemBehavior)
}
```

Transfer the velocity:

```swift
// in the gesture handle
let targetView = panGestureRecognizer.view

switch panGestureRecognizer.state {
  case .ended:
    let v = panGestureRecognizer.velocityInView(targetView.superview)
    dynamicItemBehavior.addLinearVelocity(v, forItem: targetView)
  case ...:
}
```

## Option 3: `CADisplayLink`

[`CADisplayLink`][CADisplayLink] calls you back once every frame:

- when frame is going to get rendered, you get called back to go update your app in whatever way you want
- this is how `UIDynamicAnimator` does it
- enables you to go full custom on the animation

Setup:

```
func createDisplayLink() {
  let displaylink = CADisplayLink(target: self, selector: #selector(step))
  
  displaylink.add(to: .current, forMode: .defaultRunLoopMode)
}
     
func step(displaylink: CADisplayLink) {
	// do your drawing here.
}
```

## Pros and Cons.

- `UIView.animate` pushes work to the render server, hence the application is free to do other work. `CADisplayLink` is done in the main thread.
- `UIDynamicAnimator` can create more advanced interactions (e.g., collision within views)
- `CADisplayLink` lets you completely control what to draw

## Animation to Animation

From iOS 8 all `UIView` animations will be **additive** by default.

This means that when we write:

```swift
UIView.animateWithDuration(1) { 
	// animations here
}
```

Behind the scenes, a new [`CAAnimation`][CAAnimation] instance will be created that will have its [`isAdditive`][isAdditive] property set to `true`.

Before iOS 8, `UIView.animateWithDuration` would remove the current `CAAnimation` and add a new one without any regard of where in the animation we were. This was causing our view to jump unexpectedly.

From iOS 8, calling `UIView.animateWithDuration` will add a new `CAAnimation`, and the old `CAAnimation` will continue to exist until they complete before being removed. This accomplishes a much more fluid experience.

Note that this is different than using [`UIView.animate`][animate] with [`.beginFromCurrentState`][beginFromCurrentState], as that will completely stop/remove the current `CAAnimation`, and add a new `CAAnimation` that starts from whichever state the presentation model was at that instant (this makes the view go full stop, and start a new animation from scratch, possibly towards a different direction, this is better than jumping, but still not fluid).

Supported keys for additive animations:

- `center`
- `frame`
- `bounds`
- `transform`
- `layer.transform` // only for affine transformations, where the layer parallel lines/edges are still parallel (e.g. for y rotation, but not z rotation)

Compatibility requirements for additive animations:

- No keyframe animations
- No pre-existing repeating animations
- No pre-existing absolute animations

From iOS 8, when using `UIView.animate`with `.beginFromCurrentState`, iOS will still try to use additive animations:

```swift
UIView.animateWithDuration(1,
                    delay: 0,
                  options: .beginFromCurrentState,
               animations: {
  circle.center.x = finalValue // supported, will use additive animation
  circle.alpha = 0 // not supported, will use beginFromCurrentState CAAnimation
  circle.tintColor = UIColor.redColor // not supported, will use beginFromCurrentState CAAnimation
                           },
               completion: nil)
```

### How to cancel animations

The old way of using an animation with zero duration no longer works, as it now just adds a new additive animation:

```swift
UIView.animateWithDuration(0) {
  circle.center.x = finalValue // this no longer works from iOS 8
}
```

Instead, we now need to go to the layer and remove the animations ourselves:

```swift
view.layer.removeAnimation(animation)
```

### Animation completion

With additive animations, now all completion blocks will be called when all the concurrent animations will end. 

We no longer get the completion block called with `false` when a new additive animation is added on top of our animation, instead, our completion is called when both animation end and the `isFinished` parameter will be `true`.


## Animation to Gesture

When we trigger an animation via `UIView.animate`, our animated views can catch gestures, but they are ignored by default.

To change this, we can pass [`.allowUserInteraction`][allowUserInteraction] as one of the animation options:  
when we pass this option, our view no longer catches gesture during the animation, as the view position is considered as if it's at its destination already. This is because iOS doesn't know if the thing that you're animating is something that you intend to interact with or just something that is animating in a system, but you're actually trying to interact with the thing behind it.

In other words, with `.allowUserInteraction` we're doing a model value hit test instead of a presentation value hit test.

If we want to catch gestures in the view we're animating (with `.allowUserInteraction` in the animation), we need to override the view `hitTest` and make a presentation layer hit test instead of a model layer hit test:

```swift
override func hitTest(point: CGPoint, withEvent event: UIEvent!) -> UIView! {
  let superviewPoint = convertPoint(point, toView: superview)
  let point = layer.presentationLayer.convertPoint(superviewPoint, fromLayer: superview.layer)
  return super.hitTest(point, withEvent: event)
}
``` 

Note that once we do this, all the rest of UIKit APIs will still interact with the model layer, hence when we call things such as `touch.locationInView`, they will be returned accordingly to the model layer.

### Stop Animating

To stop the animation, we need to:

1. get the current value from the presentation layer
2. set said value to our model layer
3. remove the animation

```swift
// example of stopping a transition animation
let presentationPosition = view.layer.presentationLayer().position
// note that CALayer.position is equivalent to UIView.center
view.center = presentationPosition
view.layer.removeAllAnimations()
```

If we use `UIDynamicAnimator` things are simpler:

- because `UIDynamicAnimator` doesn't have presentation and model space, we don't have this same complication of having to figure out where it is on-screen compared to where the model is or anything like that
- `UIDynamicAnimator` data is always up to date to wherever is happening in your process, and the model value is the correct position on screen right now

A way to stop animation is by removing the behavior from the view in our gesture handle:

```swift
switch (panGestureRecognizer.state) {
  case .began:
    dynamicItemBehavior.removeItem(targetView) // 👈🏻 remove behavior here
	case .ended:
    let v = panGestureRecognizer.velocityInView(targetView.superview)
    dynamicItemBehavior.addItem(targetView) // 👈🏻 add it back here
		dynamicItemBehavior.addLinearVelocity(v, forItem: targetView)
}
```

[allowUserInteraction]: https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622440-allowuserinteraction
[animate]: https://developer.apple.com/documentation/uikit/uiview/1622451-animate
[beginFromCurrentState]: https://developer.apple.com/documentation/uikit/uiview/animationoptions/1622575-beginfromcurrentstate
[isAdditive]: https://developer.apple.com/documentation/quartzcore/capropertyanimation/1412493-isadditive
[CAAnimation]: https://developer.apple.com/documentation/quartzcore/caanimation?language=objc
[animate(withDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:)]: https://developer.apple.com/documentation/uikit/uiview/1622594-animate
[UIDynamicAnimator]: https://developer.apple.com/documentation/uikit/uidynamicanimator
[CADisplayLink]: https://developer.apple.com/documentation/quartzcore/cadisplaylink
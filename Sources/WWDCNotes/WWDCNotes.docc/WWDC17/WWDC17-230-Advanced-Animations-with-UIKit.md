# Advanced Animations with UIKit

So much power has been added to animations on iOS since their inception that it's time to think about animations in a whole new way! Learn to combine and coordinate between multiple animations, resulting in interactive transitions and learn some tips and tricks along the way.

@Metadata {
   @TitleHeading("WWDC17")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc17/230", purpose: link, label: "Watch Video (32 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample code](https://github.com/cgoldsby/WWDC-2017-Session-230-Advance-Animations-with-UIKit)

## Interactive animations

An interactive animation is one in which the user's actions interactively drive the progress of your animation.

Example:

```swift
// Safe the instance of the animator
var animator: UIViewPropertyAnimator!

func handlePan(recognizer: UIPanGestureRecognizer) {
  switch recognizer.state {
    case .began:
      // create the animator on touch began
      animator = UIViewPropertyAnimator(
        duration: 1,
        curve: .easeOut,
        animations: {
          circle.frame = circle.frame.offsetBy(dx: 100, dy: 0)
        }
      )
      // pausing the animator here will produce that animation implicitly (but not trigger it),
      // we're essentially setting the speed to zero.
      animator.pauseAnimation()
      // note that while paused, the timing curve is automatically and temporaly converted to linear,
      // this makes it easy to use the animator for scrubbing
    case .changed:
      let translation = recognizer.translation(in: circle)
      // here we're scrubbing the animation
      animator.fractionComplete = translation.x / 100
    case .ended:
      animator.continueAnimation(withTimingParameters: nil, durationFactor: 0)
      // The duration factor set to zero means that the animation will pick up the original timing 
      // curve, that might be different than linear, and continue using the remaining time needed 
      // for the animation to complete based on its original duration.
  }
}
```

## New Property Animator Properties

New in iOS 11, `UIViewPropertyAnimator` has two new properties:

- [`scrubsLinearly: Bool`][scrubslinearly] // tells whether, when paused, the animator falls back to a linear timing curve or not (defaults to `true`)
- [`pausesOnCompletion: Bool`][pausesOnCompletion] // tells whether a completed animation remains in the active state (defaults to `false`)

The latter is important, because when an animator's animations finish it will automatically transition into the `.inactive` `state`. And when it does that, it releases any animations that it was previously tracking which means you cannot manipulate or even reverse them after they've finished. Thanks to this property, when set to `true`, the animator will pause at 100% `fractionComplete`, allowing you to, at any point in the future, reverse those animations.

Note that with `pausesOnCompletion` the completion block will never be called, however you can (KVO) observe the `running` property:

```swift
animator.addObserver(self, forKeyPath: "running", options: [.new], context: nil)
```

## New Property Animator Behaviors

New in iOS 11 `UIViewPropertyAnimator` also has a new behavior, which is starting as `.paused` when no animations are provided.

This will make any animations added later to run immediately without escaping:

```swift
let animator = UIViewPropertyAnimator(duration: 1, curve: .easeIn)
animator.startAnimation()

// ...

animator.addAnimations {
  // will run immediately
  circle.frame = circle.frame.offsetBy(dx: 100, dy: 0)
}
```

## Best Practices When Interrupting Springs

- when pausing an animation with springs, stop and create a new property animator (don't forget to set the current presentation value to model value) - for better fluidity in the animation
- Use critically damped spring without velocity, as these don't overshoot or oscillate
- Decompose component velocity with multiple animators, for example one for the x-axis and one for the y-axis

## View Morphing

Scaling, translation, and opacity blending of two views.

Strategy:

- `.transform: CGAffineTransform`
- Compute `transform.scale` and `transform.translation` 
- Prepare views and animate `.transform` and `.alpha`

Computing scale: this is a dimensional ratio based on the target dimension and your current dimension.

Computing translation: because of the scale, we cannot just use the delta between the two views, instead pre-apply the target view into the original view, and use that as the delta

Animate: use three animators:

1. critically damped spring for the transform
2. .easeIn for the incoming view .alpha, non linear scrubbing
3. .easeOut for the outgoing view .alpha, non linear scrubbing

```swift
func animateTransitionIfNeeded(forState state: State, duration: TimeInterval) { // ...
  let transformAnimator = UIViewPropertyAnimator(duration: duration, dampingRatio: 1) { 
    inLabel.transform = CGAffineTransform.identity
    outLabel.transform = inLabelScale.concatenating(inLabelTranslation)
  }
  // ...
  let inLabelAnimator = UIViewPropertyAnimator(duration: duration, curve: .easeIn) {
    inLabel.alpha = 1
  }
  inLabelAnimator.scrubsLinearly = false
  // ...
  let outLabelAnimator = UIViewPropertyAnimator(duration: duration, curve: .easeOut) {
    outLabel.alpha = 0
  }
  outLabelAnimator.scrubsLinearly = false
  // ...
}
```

## Tips and Tricks

- `UIView.layer.cornerRadius` is now animatable (iOS 11+):

```swift
circle.clipsToBounds = true
UIViewPropertyAnimator(duration: 1, curve: .linear) {
  circle.layer.cornerRadius = 12
}.startAnimation()
```

- New `CALayer` [`maskedCorners`][maskedCorners] property, which allows us to selectively choose which corners we want to apply our corner radius mask to

- When working with multiple animators, it's important to have the timing in sync, as it would be hard (for example) to scrub if different animators had different durations or start delays. A way to overcome this is to use UIView key frames:

```swift
// in this example buttonAnimator:
// - starts immediately and ends at 50% the original duration for the collapsed animation
// - starts at 50% the timing and end at 100% for the expanded animation
func animateTransitionIfNeeded(forState state: State, duration: TimeInterval) { // ...
  let buttonAnimator = UIViewPropertyAnimator(duration: duration, curve: .linear) { 
    // setting the duration to zero means that our keyframe animation inherits 
    // the duration of its outer property animator.
    UIView.animateKeyframes(withDuration: 0.0, delay: 0.0, options: [], animations: {
      switch state { 
        case .Expanded:
          UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.5) { 
            // Start with delay and finish with rest of animations 
            detailsButton.alpha = 1
          }
        case .Collapsed:
          UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.5) {
            // Start immediately and finish in half the time 
            detailsButton.alpha = 0
          }
      }
    }, completion: nil)
  }
}
```

[maskedCorners]: https://developer.apple.com/documentation/quartzcore/calayer/2877488-maskedcorners
[scrubslinearly]: https://developer.apple.com/documentation/uikit/uiviewpropertyanimator/2873966-scrubslinearly
[pausesOnCompletion]: https://developer.apple.com/documentation/uikit/uiviewpropertyanimator/2909004-pausesoncompletion
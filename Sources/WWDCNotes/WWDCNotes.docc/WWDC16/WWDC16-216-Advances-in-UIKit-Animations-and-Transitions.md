# Advances in UIKit Animations and Transitions

Direct onscreen manipulation is the cornerstone of the user experience on iOS. iOS 10 includes new support for making onscreen interactions even more immersive and interactive. Dive straight into the philosophy and techniques of building completely interactive, interruptible animations in your apps.

@Metadata {
   @TitleHeading("WWDC16")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc16/216", purpose: link, label: "Watch Video (46 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## [`UIViewPropertyAnimator`][UIViewPropertyAnimator]

### Features

- Familiar
- Interruptible
- Scrubbable
- Reversible
- Broad availability of timing functions 
- Running animations can be modified

![][UIViewPropertyAnimatorImage]

### Introduction

`UIViewPropertyAnimator` conform to two protocols: [`UIViewImplicitlyAnimating`][UIViewImplicitlyAnimating] and [`UIViewAnimating`][UIViewAnimating].

Thanks to these conformances, `UIViewPropertyAnimator` can be used for view controller transitions.

When creating a new animator, you're also creating a new object conforming to [`UITimingCurveProvider`][UITimingCurveProvider], which dictates the timing function that you want that animation to use.

`UIViewAnimating` protocol definition:

```swift
protocol UIViewAnimating {
  var state: UIViewAnimatingState { get }

  /// whether the animation is running or not
  var isRunning: Bool { get }

  /// whether the animation is running in the forward or reverse direction
  var isReversed: Bool { get set }
  var fractionComplete: CGFloat { get set }

  func startAnimation()
  func startAnimation(afterDelay : TimInterval)

  func pauseAnimation()

  func stopAnimation(_ withoutFinishing: Bool)
  func finishAnimation(at finalPosition: UIViewAnimatingPosition)
}
```

`UIViewImplicitlyAnimating` adds the implicit characteristics to this animator:

```swift
protocol UIViewImplicitlyAnimating {
  optional func addAnimations(_ animation: () -> Void, delayFactor: CGFloat)
  optional func addAnimations(_ animation: () -> Void)

  optional func addCompletion(_ completion: (UIViewAnimatingPosition) -> Void)

  /// Allows you to proceed from a paused animation with a completely different 
  /// finish duration, and possibly even a different timing function.
  optional func continueAnimation(
    withTimingParameters parameters: UITimingCurveProvider?, 
    durationFactor: CGFloat
  )
}
```

`UIViewPropertyAnimator` definition:

```swift
class UIViewPropertyAnimator {
  var timingParameters: UITimingCurveProvider? { get }
  var duration: TimeInterval { get }
  var delay: TimeInterval { get }
  var isUserInteractionEnabled: Bool { get set }
  var isManualHitTestingEnabled: Bool { get set }
  var isInterruptible: Bool { get set }
  
  init(
    duration: TimeInterval, 
    timingParameters parameters: UITimingCurveProvider
  )

  class func runningPropertyAnimator(
    withDuration duration: TimeInterval,
    delay: TimeInterval, options: UIViewAnimationOptions = [],
    animations: (() -> Void)?,
    completion: ((UIViewAnimatingPosition) -> Void)? = nil
  ) -> Self
}
```

### Basic usage

```swift
// define your timing function
let timing = UICubicTimingParameters(animationCurve: .easeInOut)

// create the animator
let animator = UIViewPropertyAnimator(duration: 2.0, timingParameters:timing)

// add your animations
animator.addAnimations {
  self.squareView.center = CGPoint(x: 800.0, y: self.squareView.center,y)
  self.squareView.transform = CGAffineTransform(rotationAngle: CGFloat(M_PI_2))
}

// add optional completion block to be called
animator.addCompletion {_ in
  self.squareView.backgroundColor = UIColor.orange()
}

// trigger the animation
animator.startAnimation()
```

### Pausing and reversing

The cool part of having this `UIViewPropertyAnimator` object is that now it's easy to, for example, pause the animation, reverse it, and start again:

```swift
animator.pauseAnimation()
animator.isReversed = true
/// will go backwards
animator.startAnimation()
```

Note that the completion block will be called with a [`UIViewAnimatingPosition`][UIViewAnimatingPosition] instance, telling us where we are at when the animation has completed (possible values: `.start`, `.end`, `.current`)

### Stop animating

`UIViewPropertyAnimator` also comes with `stopAnimation(_:)`, what happens is that we update the actual view model value to the animating value at that instant. 

The `withoutFinishing` parameter tells whether we should consider the animation completed or not:

- If we pass `false`, the animation is not completed and it is expected to call `finishAnimation(at:)` in the future to complete the animation (at that point the animator completion block is also called). Note that calling `finishAnimation(at:)` will not have any animation, it's up to you to make the final animation and then call `finishAnimation(at:)`

```swift
animator.stopAnimation(false)
animator.finishAnimation(.current)
// completion(.current) called
```

- If we pass `true`, the animation ends right there and the object completion block is not called

```swift
animator.stopAnimation(true)
// completion(_) not called
```

### Reversing

Three ways:

- pause animation, reverse, start animation (as we've seen in pausing chapter)
- reverse (change `isReversed` value) while the animation is running
- add a new animation to go back to the original position, for example:

```swift
animator.addAnimations {
  // go back to initial state, for example:
  view.center.x = 150.0
  view.transform = CGAffineTransform.identity
}
```

This last way is preferred as the final animation is more fluid. Note that, with this last way, the completion handler will be called with `.end` parameter, as the animation ends in the position that we just specified.

## Custom View Controller Transitions

> See Apple's article [here][appleTrans] for more.

View controller transitions basically are a bunch of interlocking protocols:

- [`UIViewControllerInteractiveTransitioning`][UIViewControllerInteractiveTransitioning] (you create this)
- [`UIViewControllerAnimatedTransitioning`][UIViewControllerAnimatedTransitioning] (you create this)
- [`UIViewControllerContextTransitioning`][UIViewControllerContextTransitioning] (iOS creates this for you)

UIkit gets the objects that conform to these protocols via a delegate: it might be a `UINavigationControllerDelegate`, or it might be a view controller `UIViewControllerTransitioningDelegate`.  
If any of these objects can return an animated transitioning object: 

- UIKit will bypass the built-in transition
- (for non-interactive transitions) UIKit will call [`animateTransition(using:)`][animateTransition(using:)] and pass a `UIViewControllerContextTransitioning` object for you to use
- (for interactive transitions) UIKit will call [`interruptibleAnimator(using:)`][interruptibleAnimator(using:)] and pass a `UIViewControllerContextTransitioning` object for you to use

[UIViewPropertyAnimatorImage]: WWDC16-216-UIViewPropertyAnimatorImage
[interruptibleAnimator(using:)]: https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1829434-interruptibleanimator
[animateTransition(using:)]: https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622061-animatetransition
[appleTrans]: https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/CustomizingtheTransitionAnimations.html
[UIViewControllerContextTransitioning]: https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning
[UIViewControllerAnimatedTransitioning]: https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning
[UIViewControllerInteractiveTransitioning]: https://developer.apple.com/documentation/uikit/uiviewcontrollerinteractivetransitioning
[UIViewAnimatingPosition]: https://developer.apple.com/documentation/uikit/uiviewanimatingposition
[UITimingCurveProvider]: https://developer.apple.com/documentation/uikit/uitimingcurveprovider
[UIViewAnimating]: https://developer.apple.com/documentation/uikit/uiviewanimating
[UIViewImplicitlyAnimating]: https://developer.apple.com/documentation/uikit/uiviewimplicitlyanimating
[UIViewPropertyAnimator]: https://developer.apple.com/documentation/uikit/uiviewpropertyanimator

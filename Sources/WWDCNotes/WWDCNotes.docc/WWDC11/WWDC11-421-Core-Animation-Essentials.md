# Core Animation Essentials

Core Animation is the layer-based animation system at the heart of the dynamic user experience seen in iOS and Mac OS X. See Core Animation in action and explore its intuitive programming model for creating compelling animations. Understand how to maintain high frame rates and learn recommended practices to deliver smooth transitions and effects in your apps.

@Metadata {
   @TitleHeading("WWDC11")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/devcenter/download.action?path=/videos/wwdc_2011__hd/session_421__core_animation_essentials.m4v", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> Check out the [Core Animation Programming Guide][capg].

> [Sample code](https://github.com/mihaelamj/CubeIn3DWithCoreAnimation)

## Architecture

![][CAArch]

## Introduction

Imagine that we'd like to move a layer from point <kbd>a</kbd> to point <kbd>b</kbd>. We'd do this by setting the `position` of the layer:

```objc
ball.position = p;
```

When we do this, Core Animation needs to take care for mainly two things for the animation to happen:

1. interpolated points
2. timing (timing function that controls how the appearance will look like)

Once these have been set, Core Animation will play the animation by producing all the intermediate frames and display them onto the device.

## Layers and layer properties

- Every `UIView` has a `CALayer` (accessible via `view.layer`)
- `UIView`'s `drawRect` renders directly into the layer (it's like we're drawing directly there)

### How to create and add new layers

```objc
#import <QuartzCore/QuartzCore.h>

CALayer* myLayer = [CALayer layer];
myLayer.bounds = CGRectMake(0,0,w,h);
myLayer.position = CGPointMake(x,y); // coordinates in the coordinate space of the hosting layer
myLayer.content = familyImage; // provide content of the layer, in this case an image
[view.layer addSubLayer:myLayer];
```

Adding/managing layers is similar to adding/managing `UIView`s, with the difference that they're much more lightweight.  
Here are some of the methods you might want to use:

- `addSublayer:`
- `insertSublayer:above:` (and similar)
- `setNeedsLayout`
- `layoutSublayers`
- `setNeedsDisplay`
- `drawInContext:` - you can override this in your `CALayer` subclass to provide your context using CoreGraphics

A more common way to override `drawInContext:` is to set the layer `delegate` and implement `drawLayer:inContext:`.

Layers support transformations is 3D (it has x, y, and z axes).

## Animating layer properties

An animation is an interpolations of points from a start position (`fromValue`) to an end position (`toValue`).

There are two styles of animations in Core Animation:

- implicit animations
- explicit animations

### Implicit animations

```objc
// This layer will fade out with a default animation duration of 1 second
myLayer.opacity = 0;
```

If we want to change the duration, we do so via a `CATransaction`:

```objc
[CATransaction setAnimationDuration:5]
myLayer.opacity = 0;
```

Note that the position of these lines is important: if `myLayer.opacity = 0;` came before the transaction duration, that animation would have used the default animation duration.

Behind the scenes:  
When we do a layer property change, if Core Animation finds a [`CAAction`][CAAction] for that property, then Core Animation will create a `CAAnimation` and adds it to the layer via `[self addAnimation:forKey:]`, which creates the implicit animation.

### Transactions

- All applied together in run loop
- `CATransaction` class
  - `duration`
  - timing function
  - `CATransaction` properties are applied at the time an implicit animation is created (e.g., `myLayer.opacity = 0;`)

Use [`[CATransaction setDisableActions:YES]`][setDisableActions] to disable animations (and get the layer property change as soon as possible).

Note that this command is only valid for the next run loop.

### Explicit animations

![][explicitAnimation]

Animations that we create, we mostly only create animations that are either (leaves in the chart above):

- `CAAnimationGroup`
- `CABasicAnimation`
- `CAKeyframeAnimation`

Example (note that for `position` we could use the implicit animation):

```objc
// First we set the final position in the model layer (explicit animations only touch the presentation layer)
layer.position = toPosition;

// Then we create the animation for that change
CABasicAnimation* drop = [CABasicAnimation animationWithKeyPath:@"position.y"];
// The initial value of the animation
drop.fromValue = [NSNumber numberWithFloat:startPosition.y];
// The final value of the animation, this should match the value we set above
drop.toValue = [NSNumber numberWithFloat:toPosition.y];
// No need to use [CATransaction setAnimationDuration:5]
drop.duration = 5;
// The animation will start as soon as we add it to the layer. 
// The key is used to identify the animation, in case we need to do some operations on it later.
[layer addAnimation:drop forKey:@"position"];
```

All implicit animations will use the name of the changed property as the animation key:

- when we write `layer.position = toPosition;`, an implicit animation with key `"position"` will be added to our layer
- by setting our explicit animation with the same key `"position"`, we're cancelling the implicit animation and make sure that only our explicit animation takes place

#### Keyframe animations

With `CAKeyframeAnimation` we're giving Core Animation a few "key" points or our animation, and leave Core Animation to figure out the rest of the interpolated points.

How to setup keyframe animations:

1. Use either
  - `path` - the animation will follow the control points of the path as its keyframes
  - `values` - the animation will interpolate among the values set here

2. Set `keyTimes` (optional) - you can tell the fraction of total time for each keyframe segment, each value has to be 0 to 1
3. Set `calculationMode` to tell which interpolation to compute for the frames between the given `values` or along the `path`, the options are:
  - Linear
  - Discrete - no interpolation, jump from key value to key value
  - Cubic - fills gaps with Bezier curves

#### Group animations

This animation gathers together other kind of explicit animations which are then:

- Applied simultaneously to layer's properties'
- Their timings are clipped to the group timing

Example:

```objc
CABasicAnimation* a1 = ....
CAKeyFrameAnimation* a2 = ...
CAAnimationGroup* group = [CAAnimationGroup animation];
group.animations = [NSArray arrayWithObjects:a1,a2,...,nil];
group.duration = ...
[layer addAnimation:group forKey:nil];
```

## How to add perspective to layers

1. Change the `zPosition` on the layers (default to `0` for all layers)
2. Set the `sublayerTransform` to a parent layer with your effect

Example:

```objc
CATransform3D perspective = CATransform3DIdentity;
perspective.m34 = -1./EYE_Z; // uses the zPosition for changing the scale of each layer

CALayer* parent = [CALayer layer];
parent.sublayerTransform = perspective;

blueLayer.zPosition = -100;
[layer addSublayer:blueLayer];

grayLayer.zPosition = 100;
[layer addSublayer:grayLayer];

// now blue and gray layer will appear to have different sizes even if their model layer 
// has the same size.
```

### Animation timing

Both `CAAnimation` and `CALayer` use the [`CAMediaTiming`][CAMediaTiming] protocol, a few properties:

- `beginTime`
- `repeatCount`, `repeatDuration` (use one or the other)
- `duration`
- `autoreverses`
- `fillMode`

### Animation notifications

- Explicit animations have a delegate:

```objc
myAnimation.delegate = self;
...

- (void)animationDidStart:(CAAnimation *)theAnimation {
  ...
}

- (void)animationDidStop:(CAAnimation *)theAnimation finished:(BOOL)flag {
  ...
}
```

- Implicit animations can have a completion block via `CATransaction`:

```objc
// Always declare the transaction before the implicit animation
[CATransaction setCompletionBlock:^{
  // block that runs when animations have completed
  [CATransaction setDisableActions:YES];
  [layer removeFromSuperlayer];
}];

layer.opacity = 0;
layer.position = CGPointMake (2000, layer.position.y);
```

### Presentation vs model

- Layer properties do not reflect active animations
- Use `-presentationLayer` method to get screen values
  - Creates a temporary layer with animations applied
  - Asking for sublayers returns presentation versions

- Useful for from values of animations

```objc
anim = [CABasicAnimation animationWithKeyPath:@”borderColor”];
anim.fromValue = [[layer presentationLayer] borderColor];
```

- And for hit-testing against real geometry

```objc
hitLayer = [[[layer presentationLayer] hitTest:p] modelLayer];
```

## Replicators

- [`CAReplicatorLayer`][CAReplicatorLayer] creates a specified number of sublayer copies with varying geometric, temporal, and color transformations.
- You control the number of copies via its `instanceCount` property
- by default all copies will sit on top of each other with all the same properties, we can change that by setting one (or more) of the following `CAReplicatorLayer` properties (note that they're all also animatable):
  - `instanceTransform: CATransform3D`, `instanceColor: CGColor?`, `instanceRedOffset: Float`, `instanceGreenOffset: Float`, `instanceBlueOffset: Float`, `instanceAlphaOffset: Float` 

## Particles

![][CAEmitterLayerImg]

- done thanks to [`CAEmitterLayer`][CAEmitterLayer]
- can use them from iOS 5 and later
- can be used to mock water, smoke, fire, etc

How to:

- set the `emitterShape` property to set the shape of the emitter (line, circle etc)
- set the `emitterPosition` property to set the center of the particle emitter
- set the `emitterMode` property to specify where the particles are emitted from
- set the `emitterCells` property with an array of [`CAEmitterCell`][CAEmitterCell], which represent characteristics of your particles (how do they look and behave)

[CAArch]: WWDC11-421-CAArch
[explicitAnimation]: WWDC11-421-explicitAnimation
[CAEmitterLayerImg]: WWDC11-421-CAEmitterLayerImg
[CAEmitterCell]: https://developer.apple.com/documentation/quartzcore/caemittercell
[CAEmitterLayer]: https://developer.apple.com/documentation/quartzcore/caemitterlayer
[CAReplicatorLayer]: https://developer.apple.com/documentation/quartzcore/careplicatorlayer
[CAMediaTiming]: https://developer.apple.com/documentation/quartzcore/camediatiming
[CAAction]: https://developer.apple.com/documentation/quartzcore/caaction
[setDisableActions]: https://developer.apple.com/documentation/quartzcore/catransaction/1448261-setdisableactions?language=objc
[capg]: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514-CH1-SW1
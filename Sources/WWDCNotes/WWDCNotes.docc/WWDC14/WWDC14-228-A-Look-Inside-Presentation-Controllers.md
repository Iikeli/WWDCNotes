# A Look Inside Presentation Controllers

iOS 8 brings you powerful new means of presenting content within your apps. Hear how presentation controllers were leveraged by UIKit to give you fine grain control using new alert and searching APIs. Dive deep into how presentation controllers work and how you can use them to present content within your app in exciting new ways.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/228", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Official sample code](https://developer.apple.com/library/archive/samplecode/LookInside/Introduction/Intro.html), [working sample code](https://github.com/qrush/LookInside)

## Presentation basics

- <kbd>presented</kbd> view controller: the view controller we're showing
- <kbd>presenting</kbd> view controller the view controller that is presenting the presented view controller
- <kbd>content</kbd>: in [`UIPresentationController`][UIPresentationController] terms, this the <kbd>presented</kbd> content, it's the foreground stuff the user is meant to interact with
- <kbd>chrome</kbd>: in [`UIPresentationController`][UIPresentationController] terms, this is the background content (behind the <kbd>content</kbd>) that is usually dimmed

`UIPresentationController` manages the view controllers presentation in your app. All UI view controller presentations in iOS 8 are backed by `UIPresentationController`.

Since `UIPresentationController` owns/can provide <kbd>chrome</kbd>, it can also provide its own animations for that chrome and can also animate its chrome alongside your existing animator objects' custom animations.

### Division of responsibilities

Before iOS 8, your animator object was responsible to: 

- animate the controller content
- positioning the content
- manage the chrome

From iOS 8, the content positioning and chrome management are `UIPresentationController`'s responsibility.

`UIPresentationController` also drives adaptation in your application, as it knows about the presentation traits.

## UIKit presentations

Let's say that we want to make a custom transition like the following:

<video autoplay muted loop style="max-width: 100%;">
  <source src="WWDC14-228-dimTransition">
</video>

In iOS 7, UIKit introduced the `transitioningDelegate` ([`UIViewControllerTransitioning`][UIViewControllerTransitioning]) that was responsible to provide an object animator for the custom presentation animation. From iOS 8 `transitioningDelegate`, also provides the presentation controller via a new method:

```objc
- (UIPresentationController *)presentationControllerForPresentedViewController:(UIViewController *)presented 
                                                      presentingViewController:(UIViewController *)presenting 
                                                          sourceViewController:(UIViewController *)source;
```

How to:

1. In both the <kbd>presenting</kbd> and <kbd>presented</kbd> view controller we will set a custom `transitionDelegate`

```objc
transitionDelegate = [[OverlayTransitioningDelegate alloc] init];
[overlayViewController setTransitioningDelegate:transitionDelegate];
```

2. In the <kbd>presented</kbd> view controller we will set the modal presentation to `.custom`. This indicates to UIKit that we should consult your transitioning delegate for a custom Presentation Controller to use for the presentation:

```objc
[self setModalPresentationStyle:UIModalPresentationCustom];
```

3. In our transition delegate, we need to provide that `UIPresentationController`:

```objc

- (UIPresentationController *)presentationControllerForPresentedViewController:(UIViewController *)presented 
                                                      presentingViewController:(UIViewController *)presenting
                                                          sourceViewController:(UIViewController *)source
{
  return ...
}
```

4. In our custom `UIPresentationController`, we need to add the animations for the dimming view.

First in the `presentationTransitionWillBegin`:

```objc
- (void)presentationTransitionWillBegin
{
  UIView* containerView = [self containerView];
  UIViewController* presentedViewController = [self presentedViewController];
  // we make our dimming view full screen
  [dimmingView setFrame:[containerView bounds]];
  // we make our dimming view fully transparent
  [dimmingView setAlpha:0.0];

  // we add our dimming view above all the other content in the presentation
  [containerView insertSubview:dimmingView atIndex:0];
  
  // we add the dimming view fade-in along with the rest of the coordinator animations
  [[presentedViewController transitionCoordinator] animateAlongsideTransition:^(id<UIViewControllerTransitionCoordinatorContext>context) {
    [dimmingView setAlpha:1.0];
  } completion:nil];
}
```

Then in `dismissalTransitionWillBegin`:

```objc
- (void)dismissalTransitionWillBegin
{
  // here we just fade-out the dimming view
  [[[self presentedViewController] transitionCoordinator] animateAlongsideTransition:^(id<UIViewControllerTransitionCoordinatorContext>context) {
      [dimmingView setAlpha:0.0];
  } completion:nil];
}
```

5. In our transition delegate we need to provide our custom animator object for both presenting and dismissing our view:

```objc
- (id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented
                                                                   presentingController:(UIViewController *)presenting
                                                                      sourceController:(UIViewController *)source
{
  OverlayAnimatedTransitioning *animationController = [[OverlayAnimatedTransitioning alloc] init];
  return animationController;
}

- (id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed
{
  OverlayAnimatedTransitioning *animationController = [[OverlayAnimatedTransitioning alloc] init];
  return animationController;
}
```

6. The sidebar width in the animation above is about a third of the <kbd>presenting</kbd> view controller, in order to achieve this, we need to implement various `UIPresentationController` methods:

```objc
// Note that this is one of the methods coming from UIContentContainer. 
// This method is also here because the presenting view controller is not a container for the Presentation Controller. 
// The size that gets passed to the presentation view controller is the same size that is passed to the presenting view controller.
// The Presentation Controller is the container for the presented controller, hence it decides what is the size for the 
// presented view controller.
- (CGSize)sizeForChildContentContainer:(id <UIContentContainer>)container withParentContainerSize:(CGSize)parentSize {
  return CGSizeMake(floorf(parentSize.width / 3.0), parentSize.height);
}

// This is where we return to the view controller transitioning system what frame we'd like the presented view to have.
- (CGRect)frameOfPresentedViewInContainerView {
  CGRect presentedViewFrame = CGRectZero;
  CGRect containerBounds = [[self containerView] bounds];
  presentedViewFrame.size = [self sizeForChildContentContainer: (UIView<UIContentContainer> *)[self presentedView]
  withParentContainerSize:containerBounds.size];
  presentedViewFrame.origin.x = containerBounds.size.width - presentedViewFrame.size.width;
  return presentedViewFrame;
}
```

7. To add rotation support, all we need to do is implement in our presentation controller the `containerViewWillLayoutSubviews` method:

```objc
- (void)containerViewWillLayoutSubviews
{
  [dimmingView setFrame:[[self containerView] bounds]];
  [[self presentedView] setFrame:[self frameOfPresentedViewInContainerView]];
}
```

[UIViewControllerTransitioning]: https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate
[UIPresentationController]: https://developer.apple.com/documentation/uikit/uipresentationcontroller
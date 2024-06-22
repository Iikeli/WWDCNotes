# View Controller Advancements in iOS 8

View controllers are fundamental to creating apps on iOS. Learn about the enhancements made to view controllers in iOS 8 to improve the user experience in your apps. Dive into using and creating transition coordinators and find out about all-new additions to split view controllers and navigation controllers.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/214", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Support for Adaptive User Interfaces

Before iOS 8, most of your application interface/structure was determined by looking at:

- Device type
- Interface Orientation
- Size

From iOS 8, the first two have been reshaped into traits and trait collection. The new way to structure the app is via:

- Traits and trait collection
- Size

### What's a trait collection?

A collection of traits, more specifically:

- horizontalSizeClass
- verticalSizeClass
- userInterfaceIdiom
- displayScale

What's a (horizontal/vertical) size class? A trait that coarsely defines the space available for that axis (possible values: `.compact` or `.regular`)

Most UIKit objects conform to a new [`UITraitEnvironment`][UITraitEnvironment] that both provide the current trait collection and also lets us get notified when the trait environment changes:

```objc
@protocol UITraitEnvironment <NSObject>
  @property UITraitCollection *traitCollection;
  - (void)traitCollectionDidChange:
@end
```

A parent view controller can override the trait collection for its child. For example, a `UISplitViewController` might give different traits to its primary and secondary view controllers.

How to:

```objc
@interface UIViewController <UITraitEnvironment>
  - (void)setOverrideTraitCollection: forChildViewController:
  - (UITraitCollection *)overrideTraitCollectionForChildViewController:
@end
```

## Presentation Controllers

For finer control on custom transitions, from iOS 8 we have `UIPresentationController`, which takes care of calling and retaining references to various parts of each transition (e.g., the `containerView`, the `presentedView`). Note that the presented view might not be the presented view controller view, as UIKit might add a dimming view/drop shadow around that view because that's what the custom presentation demanded.

To overcome this `UIViewControllerContextTransitioning` now offers a new `view(forKey:)` besides the old `viewController(forKey:)`, make sure to use this new function to figure out which views are actually participating in the animation.

```objc
@protocol UIViewControllerContextTransitioning
  - (UIView *)viewForKey:(NSString *)key AVAILABLE_IOS(8_0);
@end

@interface UIPresentationController : NSObject
          <UIAppearanceContainer, UITraitEnvironment, UIContentContainer>

  @property(nonatomic, readonly) UIView *containerView;
  - (UIView *)presentedView;

  // we can decide whether we want the removal of the presenter views
  - (BOOL)shouldRemovePresentersView;

  // we can also decide whether the presented view will take over the screen.
  // If you set this to NO, your presentation will no longer adapt.
  - (BOOL)shouldPresentInFullScreen;
@end
```

Previous iPad-only styles are available on the iPhone (by default they adapt to full screen presentations when the horizontal trait is `.compact`).

New Presentation Styles:

- `UIModalPresentationOverFullscreen`
- `UIModalPresentationOverCurrentContext`
- `UIModalPresentationPopover`

All presentation styles have an associated presentation controller that we can access to (to set their delegate):

```objc
-[UIViewController presentationController]
-[UIViewController popoverPresentationController]
```

Thanks to the [`UIAdaptivePresentationControllerDelegate`][UIAdaptivePresentationControllerDelegate] delegate we can 

```
@protocol UIAdaptivePresentationControllerDelegate <NSObject>

  @optional
  // here for example we can tell the presentation to not adapt to full screen when the horizontal trait is compact
  - (UIModalPresentationStyle)adaptivePresentationStyleForPresentationController:

  // allows you to return a whole new view controller that should be presented in that style.
  - (UIViewController *)presentationController:viewControllerForAdaptivePresentationStyle:
@end
```

## Transition Coordinators

- Objects conforming to [`UIViewControllerTransitionCoordinator`][UIViewControllerTransitionCoordinator]
- Every transition coordinator has an associated transition
- This coordinator can add animation blocks to the original animation
- from iOS 8, transition coordinators also conform to [`UIContentContainer`][UIContentContainer] (`UIViewController`s also conform to this)

```objc
@protocol UIContentContainer <NSObject>
  @property (nonatomic, assign) CGSize preferredContentSize;
  - (void)preferredContentSizeDidChangeForChildContentContainer:
  - (void)systemLayoutFittingSizeDidChangeForChildContentContainer:
  - (void)sizeForChildContentContainer:withParentContainerSize:
  - (void)willTransitionToTraitCollection:withTransitionCoordinator:
  - (void)viewWillTransitionToSize:withTransitionCoordinator:
@end
```

Use `- (void)willTransitionToTraitCollection:withTransitionCoordinator:` to react to resizing (replaces of old deprecated rotation callbacks).

When a trait changes, you can check [`UIViewControllerTransitionCoordinatorContext`][UIViewControllerTransitionCoordinatorContext]'s `targetTransform` to participate in the rotation animation (if there's a rotation).
```
@protocol UIViewControllerTransitionCoordinatorContext
  - (CGAffineTransform)targetTransform;
  - (UIView *)viewForKey:(NSString *)key;
@end
```

For example:

```objc
- (void) viewWillTransitionToSize:(CGSize)s
         withTransitionCoordinator:(UIVCTC)t {
  orientation = [self orientationFromTransform: [t targetTransform]]; 
  oldOrientation = [[UIApplication sharedApplication] statusBarOrientation];

  [self myWillRotateToInterfaceOrientation:orientation duration: duration];

  [t animateAlongsideTransition:^(id <UIVCTCContext>) {
    [self myWillAnimateRotationToInterfaceOrientation:orientation
                                             duration:duration];
  }
  completion: ^(id <UIVCTCContext>) {
    [self myDidAnimateFromInterfaceOrientation:oldOrientation];
  }];
}
```

Note:

- The legacy rotation methods are still available (just don’t implement the new methods)
- Most view controller transitions are immediate when called from within the dynamic scope of this method (e.g., if you do a push here, the push will be done immediately without animation)
- Call `super` in order to forward to descendant view controllers
- Only necessary when you need to do a special size transition

[UIAdaptivePresentationControllerDelegate]: https://developer.apple.com/documentation/uikit/uiadaptivepresentationcontrollerdelegate
[UIViewControllerTransitionCoordinatorContext]: https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinatorcontext
[UIContentContainer]: https://developer.apple.com/documentation/uikit/uicontentcontainer
[UIViewControllerTransitionCoordinator]: https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinator
[UIPresentationController]: https://developer.apple.com/documentation/uikit/uipresentationcontroller
[UITraitEnvironment]: https://developer.apple.com/documentation/uikit/uitraitenvironment
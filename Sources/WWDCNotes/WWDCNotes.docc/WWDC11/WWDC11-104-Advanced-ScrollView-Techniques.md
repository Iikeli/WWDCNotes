# Advanced ScrollView Techniques

Come learn about how to achieve the appearance of infinite scrolling in either one or two dimensions. We'll also look at how to change the resolution of drawn content during zooming, without requiring the use of CATiledLayer.

@Metadata {
   @TitleHeading("WWDC11")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/devcenter/download.action?path=/videos/wwdc_2011__hd/session_104__advanced_scroll_view_techniques.m4v", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Basics

- To enable scrolling your content in a `UIScrollView`, set its `contentSize`, which tells the `UIScrollView` how much content there is.
- To know what portion of the content is currently shown on screen, use `UIScrollView`'s `contentOffset`, which represents the top left current visible point on the scroll view frame (a.k.a. The point at which the origin of the content view is offset from the origin of the scroll view).
- To get zooming working on a scroll view:
  1. create a type conforming to `UIScrollViewDelegate`, implement [`viewForZooming(in:)`][viewForZooming(in:)]
  2. set an instance of this type as your `UIScrollView` instance `delegate`
  3. set the `minimumZoomScale` and the `maximumZoomScale` in your `UIScrollView` instance to be different (both are `1.0` by default)

## Advanced Techniques

1. Infinite scrolling
2. Stationary views 
3. Custom touch handling 
4. Redraw after zooming

### 1. Infinite scrolling

> 📚 [Download `StreetScroller` code sample][cs1].

The user can keep scrolling in one direction and never hit the edge of the content (e.g. a photo carousel that automatically wraps).

How to achieve this:

1. make the `contentSize` about twice the size of what's visible on screen
2. when the user is about to hit the content edge, adjust the `contentOffset` to go into the middle of the `contentSize` (a.k.a the scrollable area)
3. adjust the frames of our content subviews to the same amount as the `contentOffset` so that they're still centered in the visible content area

The last two steps needs to be done concurrently and the user won't be able to notice.

Where to implement this: the idea is to re-layout those subviews every time the user scrolls. 

We have two possible ways:

- sub-class `UIScrollView`, and override the `layoutSubviews()` method (the WWDC session uses this one). `layoutSubviews()` is called at every frame of zooming and scrolling (a.k.a. anytime the scroll view bounds change)
- use `UIScrollViewDelegate`'s [`scrollViewDidScroll(_:)`][scrollViewDidScroll(_:)]

In `layoutSubviews()` we will:

- call `UIScrollView`'s `setContentOffset(_:animated:)`, to shift the content back
- set `UIView`'s `center` of `frame`, to shift subviews by the same amount as the scroll view content

Code for infinite horizontal scroll view:

```objc
@implementation InfiniteScrollView

// Recenter content periodically to achieve impression of infinite scrolling
- (void)recenterIfNecessary
{
    CGPoint currentOffset = [self contentOffset];
    CGFloat contentWidth = [self contentSize].width;
    CGFloat centerOffsetX = (contentWidth - [self bounds].size.width) / 2.0;
    CGFloat distanceFromCenter = fabs(currentOffset.x - centerOffsetX);
    
    // We re-center when the offset is greater than 25% off the center, this is arbitrary.
    if (distanceFromCenter > (contentWidth / 4.0)) {
        self.contentOffset = CGPointMake(centerOffsetX, currentOffset.y);
        
        // Move content by the same amount so it appears to stay still
        for (UILabel *label in self.visibleLabels) {
            CGPoint center = [self.labelContainerView convertPoint:label.center toView:self];
            center.x += (centerOffsetX - currentOffset.x);
            label.center = [self convertPoint:center toView:self.labelContainerView];
        }
    }
}

- (void)layoutSubviews
{
    [super layoutSubviews];
    [self recenterIfNecessary];
}

...

@end
```

### 2. Stationary views

Views that remain pinned in place in one dimension/direction, but scroll with the scrolling content on the other axis. Think like headers and footers.

This can be the case where we have one piece of the content that should not zoom or scroll along with the rest of the content (e.g. the title of an image).

In the session they implement a scenario where:

- there's an image title that sticks on top of the scroll view
- the image can be scrolled and zoomed, the title stays in place
- the title only disappears when the user scrolls down on the image, so that the image can be seen in full
- any other interaction (zoom or scroll will make the title reappear)

Configuration:  
We have one scroll view that takes the whole available space, which has two subviews:

1. the header/title view, which doesn't zoom
2. the `UIImageView` that can be zoomed, this is the view returned in [`viewForZooming(in:)`][viewForZooming(in:)]

What to do next:

- Since only our `UIImageView` can scroll, the first thing we need to do is to make sure that our header view stays centered horizontally when we zoom/scroll in the image view. This is done by setting the header `frame.origin.x` to be equivalent to `contentOffset.x` in `layoutSubviews()`.
- when we zoom in a scroll view, the scroll view content size is automatically updated to the zoomed size of the view returned in `viewForZooming(in:)`. In our case, we also need the scroll view to consider the header view size, hence we need to override the `contentSize` setter to also consider the header.

### 3. Custom touch handling

> The session focuses on adding multi-touch handlers to subviews of the scroll view.

You can get the `UIScrollView`'s pan and pitch gesture recognizers via the [`panGestureRecognizer`][panGestureRecognizer] and [`pinchGestureRecognizer`][pinchGestureRecognizer] properties. These are the same recognizers tat `UIScrollView` uses to manage its own gestures (for scrolling and zooming, respectively).

In this session they implement a scenario where:

- swiping up/down from the bottom of the scroll view will make another view appear/disappear instead of scrolling in the scroll view

Implementation (you can add this code in `loadView()`/`viewDidLoad()`:

```objc
UIScrollView *scrollView = [self scrollView]:
UISwipeGestureRecognizer *swipeUp = [[UISwipeGestureRecognizer alloc] initWithTarget:self action: @selector (handleSwipeUp:)];
swipeUp.direction = UISwipeGestureRecognizerDirectionUp;

[scrollView addGestureRecognizer: swipeUp];
// 👇🏻 this is required, otherwise the scrollView.panGestureRecognizer would always trigger before our swipe up
[scrollView.panGestureRecognizer requireGestureRecognizerToFail:swipeUp];
```

Note that this implementation makes the scroll view pan gesture to wait to trigger because it needs to make sure the gesture is not a swipe. However, in our case we want this behavior only for the bottom of the scroll view and not the whole screen. So we can limit the target area:

```objc
- (BOOL) gestureRecognizer: (UIGestureRecognizer *)gestureRecognizer
        shouldReceiveTouch: (UITouch *) touch
{
  UIScrollView #scrollView = [self scrollView];
  CGRect  visibleBounds = [scrollView bounds]:
  CGPoint touchPoint = [touch locationInView:scrollView];
  if (touchPoint.y < CGRectGetMaxY(visibleBounds) - 75)
    return NO:
  return YES;
}
```

### 4. Redraw after zooming

> 📚 [Download `ScrollViewSuite` code sample][cs2]. [Download `PhotoScroller` code sample][cs3].

> The session focuses on small bits of content that need to be redrawn once the user is done zooming.

The idea is that we content gets zoomed in, it starts to get blurry, so you want to redraw it to make it crisp once again.

We do this redraw only after the zoom has ended, not while the user is zooming (the reason being this operation is expensive and we'd discard things constantly as the user keeps zooming): this can be obtained via [`scrollViewDidEndZooming(_:with:atScale:)`][scrollViewDidEndZooming(_:with:atScale:)], which also lets us know what is the final scale.

How-to (⚠️ only for small pieces of content):

```objc
- (void)scrollViewDidEndZooming: (UIScrollView *) sv
                       withView: (UIView *) view
                       atScale: (float)scale
{
  scale *= [[[scrollView window] screen] scale]:
  [view setContentScaleFactor:scale]:
}
```

The [`contentScaleFactor`][contentScaleFactor] of a view is essentially a multiplier applied to the bound size of the view used to determine how big your view rect backing storage should be.

[contentScaleFactor]: https://developer.apple.com/documentation/uikit/uiview/1622657-contentscalefactor
[scrollViewDidEndZooming(_:with:atScale:)]: https://developer.apple.com/documentation/uikit/uiscrollviewdelegate/1619407-scrollviewdidendzooming
[cs3]: https://developer.apple.com/library/archive/samplecode/PhotoScroller/Introduction/Intro.html#//apple_ref/doc/uid/DTS40010080-Intro-DontLinkElementID_2
[cs2]: https://developer.apple.com/library/archive/samplecode/ScrollViewSuite/Introduction/Intro.html#//apple_ref/doc/uid/DTS40008904
[viewForZooming(in:)]: https://developer.apple.com/documentation/uikit/uiscrollviewdelegate/1619426-viewforzooming
[scrollViewDidScroll(_:)]: https://developer.apple.com/documentation/uikit/uiscrollviewdelegate/1619392-scrollviewdidscroll
[viewForZooming(in:)]: https://developer.apple.com/documentation/uikit/uiscrollviewdelegate/1619426-viewforzooming
[cs1]: https://developer.apple.com/library/archive/samplecode/StreetScroller/Introduction/Intro.html#//apple_ref/doc/uid/DTS40011102-Intro-DontLinkElementID_2
[panGestureRecognizer]: https://developer.apple.com/documentation/uikit/uiscrollview/1619425-pangesturerecognizer
[pinchGestureRecognizer]: https://developer.apple.com/documentation/uikit/uiscrollview/1619381-pinchgesturerecognizer
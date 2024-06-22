# Sharing code between iOS and OS X

Learn what the iWork engineers did to ship iWork for iOS and Mac from a single codebase. Explore the patterns for sharing code between desktop and mobile, and see how you can optimize your code and write great apps.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/233", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What code can we share?

- ✅ model code: no conversion nor data loss when shared across platform
- ❌ view code: different gestures, HIG, display size, flows
- ❓ controller code: it depends on the kind of controllers

### Why not shared view code

We could use shimming:

```objc
#if TARGET_OS_IPHONE
@interface MyAwesomeView : UIView
#else
@interface MyAwesomeView : NSView
#endif
{
  ...
} @end
```

However `UIView` and `NSView` have many different API and behaviors:

| `UIView` | `NSView` |
| --- | --- |
| Receives and handles events | Receives and handles events |
| Responsible for drawing | Responsible for drawing |
| Always backed by Core Animation | Layer-backed views optional |
| Layer Origin in top left | Origin in bottom left |
| Subviews can draw outside view bounds | Subviews clip to view bounds |
| Gesture Recognizers | Mouse event handling |
| Animation APIs | Drag and Drop |
| | Tooltip support |

Cons: 

- Commonly breaks builds
- Hard to target fixes
- Requires `#if TARGET_OS_IPHONE` by design
  - Hard to read
  - Hard to maintain

Behaviors and UI will take on the look of the original (development) platform by default.

### Why sharing controller code has not a definitive answer

- Similar to views, we cannot share view controllers that handle user interactions, hotkeys, mouse handling.
- We can share common controller logic and model controllers:
  - split the view controller in various components
  - share only what can be shared

## Using Frameworks to share more code Shared rendering

- `NSView` has a [`isFlipped`][isFlipped] property that you can use to have the same core graphics coordinates as in iOS (origin on top left corner)

- Create an image wrapper around `UIImage` and `NSImage` so that you can grab and share the same <kbd>CoreGraphics</kbd> image model on both platforms (accessed via `UIImage` and `NSImage` [`cgImage`][cgimage] property.)
  - Use this kind of wrappers for just simple objects, not view controllers

## Performance

As different platforms have different resources available, creating a cross-platform app is also a great opportunity for performance improvements.

Lazily loaded model:

- Make sure that sections of model are self-contained
- Only load what the user needs
- Load in parallel

This kind of gains results in faster opening of documents that benefit all platforms.

## Data versioning

Users will not update all apps at the same time, make sure old versions of the app can still open data models created in newer versions, and viceversa.

## Cross-platform projects in Xcode

### Targets

Create a separate target for each platform.

A target:

- Defines a single product to build 
- Organizes inputs into build system
- Is owned by projects (<kbd>.xcodeproj</kbd>)

### Libraries

Use libraries for shared code.

Static libraries are:

- Built with the project
- Included as part of the executable

Dynamic libraries are:

- Optionally built with the project (no need to build them again until you change them)
- Excluded from the executable (they're another mach-O file in your app package)

### Configuration files

Use Xcode Configuration Files for different targets. See what you can set here in the [Build Setting Reference guide][bsrg].

Used Xcode Configuration inheritance to share common configurations among platforms:

```
// In the iOS specific .xcconfig
#include "Common.xcconfig"
...
```

[bsrg]: https://developer.apple.com/library/archive/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html
[cgimage]: https://developer.apple.com/documentation/appkit/nsbitmapimagerep/1395506-cgimage
[isFlipped]: https://developer.apple.com/documentation/appkit/nsview/1483532-isflipped
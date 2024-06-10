# Eliminate animation hitches with XCTest

Animations can dramatically enhance the user experience of your app, provide a sense of direct manipulation, and help people to better understand the results of their actions. Animation hitches can break that experience. Discover how to use XCTest to detect interruptions to smooth scrolling and animations, and learn how to catch regressions before they affect the people relying on your app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10077", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



## Hitches
* Scrolling animation sometimes appears to jitter, those jitters in Xcode are called hitches
* A _hitch_ is a frame that is displayed on the screen later than expected. 
  ![][image-1]

* iPhones and iPads have screens with 60Hz refresh rate, iPad Pros go up to 120Hz. 60Hz means that each frame should stay on screen for 16.67ms, 120Hz means 8.33ms
* VSYNC is what usually manages swapping frames on screen, we see a hitch when a frame misses its expected VSYNC 

## Hitch Metrics
* Hitch time: how many ms a frame is late
* Hitch ratio: Hitch time in ms per second for a given duration of hitches for a given duration (for a test, scroll event or transition)
* Hitch time isn’t accurate as the desired screen refresh rate isn’t always 60 or 120fps, it can be perfectly fine for VSYNC to swipe zero frames for an idle screen
* This is why we use Hitch ratio, it’s consistent across different cases and tests

### User impact targets for hitch ratio:
![][image-2]

> You should always aim to have \<= 5ms/s hitch ratio

## Measuring Hitches
* It’s possible to measure hitches in development and production environment (production for iOS 14 only) using different tools:  
  ![][image-3]

* XCTest Metrics is used to measure hitches in development environment, while MetricKit and Xcode Organizer can show you measures from the customers devices directly
* XCTest Metrics has many tests you can use to measure different parts of your application, the one we’re concerned with here is `XCTOSSignPostMetric`
* `XCTOSSignPostMetric` used to measure animation duration in previous versions of Xcode, new in Xcode 12, this metric will provide 5 additional information:
	* Total count of hitches
	* Total duration of hitches
	* Hitch time ratio
	* Frame rate
	* Frame count

* To collect these metrics, you should write a test case that collects `os_signpost` intervals with the new 
* In Xcode 11, this produced a non-animation `os_signpost` intervals:

```swift
os_signpost(.begin, log: logHandle, name: "performInterval")
os_signpost(.end, log: logHandle, name: "performInterval")
```

* Now in Xcode 12, you can use the `.animationBegin` instead to specify the extra metrics related to animation:

```swift
os_signpost(.animationBegin, log: logHandle, name: "performAnimationInterval")
os_signpost(.end, log: logHandle, name: "performAnimationInterval")
```

* You can also use one of the predefined `UIKit` instrumented intervals for testing around navigation transitions and scrolling, these are sub-metrics provided on the XCTOSSignpostMetric class.

```swift
extension XCTOSSignpostMetric {
	 open class var navigationTransitionMetric: XCTMetric { get }
	 open class var customNavigationTransitionMetric: XCTMetric { get }
	 open class var scrollDecelerationMetric: XCTMetric { get }
	 open class var scrollDraggingMetric: XCTMetric { get }
}
```

* Here is a sample test case that launches the app, taps on _”Meal Planner"_ and measures the scrolling animation in a `.fast` velocity:

```swift
// Measure scrolling animation performance using a Performance XCTest
func testScrollingAnimationPerformance() throws {
	app.launch()
	app.staticTexts["Meal Planner"].tap()
	let foodCollection = app.collectionViews.firstMatch

	measure(metrics: [XCTOSSignpostMetric.scrollDecelerationMetric]) {
	foodCollection.swipeUp(velocity: .fast)
	}
}
```

* To avoid swiping between different content in the 5 iterations of the measure block, we can reset the application state during measurements:

```swift
func testScrollingAnimationPerformance() throws { 
	app.launch()
	app.staticTexts["Meal Planner"].tap()
	let foodCollection = app.collectionViews.firstMatch

	let measureOptions = XCTMeasureOptions()
	measureOptions.invocationOptions = [.manuallyStop]

	measure(metrics: [XCTOSSignpostMetric.scrollDecelerationMetric],
		options: measureOptions) {
	foodCollection.swipeUp(velocity: .fast)
	stopMeasuring()
	foodCollection.swipeDown(velocity: .fast)
	}
}
```

#### Tips for creating performance tests:
* Create a separate test scheme for your Performance XCTest
* Use `Release`  Build Configuration and uncheck Debug executable
* Switch off Automatic Screenshots and Code Coverage
* Turn off all diagnostic options, checkers, sanitizers and memory management

[image-1]:	frame_delay.png
[image-2]:	hitch_ratio_targets.png
[image-3]:	hitch_measurement_tools.png

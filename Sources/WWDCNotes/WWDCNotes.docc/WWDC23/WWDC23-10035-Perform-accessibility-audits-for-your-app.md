# Perform accessibility audits for your app

Discover how you can test your app for accessibility with every build. Learn how to perform automated audits for accessibility using XCTest and find out how to interpret the results. We’ll also share enhancements to the accessibility API that can help you improve UI test coverage.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10035", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(fbernutz)
      @GitHubUser(multitudes)
   }
}



![Sketchnote of WWDC 2023 talk about how to perform accessibility audits for your app][sketchnote]

[sketchnote]: sketchnote.jpg

# Chapters
[0:40 - Discover accessibility audits](https://developer.apple.com/videos/play/wwdc2023/10035/?time=40)  
[2:52 - Add audits to your UI tests](https://developer.apple.com/videos/play/wwdc2023/10035/?time=172)  
[9:34 - Filter audit issues](https://developer.apple.com/videos/play/wwdc2023/10035/?time=574)  
[11:41 - Considerations when running audits](https://developer.apple.com/videos/play/wwdc2023/10035/?time=701)  
[12:59 - Expose elements hidden from accessibility to UI tests](https://developer.apple.com/videos/play/wwdc2023/10035/?time=779)

## Intro 
- Accessibility audits. Perform automated accessibility audits in your UI tests.  
- Automation elements. Expose elements for a great testing and a accessibility experience.

![testing and a accessibility][testingAndAccessibility]

[testingAndAccessibility]: testingAndAccessibility.jpg

# Accessibility audits
By writing tests, we're able to catch and fix bugs before we ship code. It's how we ensure the quality of the product. And an accessible product is a high-quality product.

Xcode ships with a tool called the Accessibility Inspector. This tool provides an easy way to find, diagnose, and fix accessibility issues within your app. 

![Accessibility inspector][inspector]

[inspector]: inspector.jpg

The Inspector can audit individual views in your app for common accessibility issues.

## The app
This is my sample app. It has two tabs. The first tab provides me with motivational quotes, and the second lets me write down my thoughts for self-reflection. In the quote tab, I have a text view which displays the quote. And this text view is placed on top of a background image. There's also a New Quote button which refreshes the quote.

![Accessibility inspector][app]

[app]: app.jpg

I can launch the Accessibility Inspector and perform an audit of my app. The Inspector checks for all kinds of issues, like providing sufficient element descriptions and ensuring proper contrast. And the issues it finds are displayed in a table with detailed descriptions about each issue.

![Accessibility inspector][inspector2]

[inspector2]: inspector2.jpg

Accessibility audits are automatable. You are now able to perform audits in your UI tests.  
Calling performAccessibilityAudit on your XCUIApplication will audit the current view for accessibility issues just as the Inspector does. There's no need for assertions: if any issues are found, your test automatically fails.
```swift
// Test for accessibility issues

func testAccessibility() throws {
	let app = XCUIApplication()
	app.launch ()
	
	try app.performAccessibilityAudit()
}
```
Let's dive into a quick demo to see this in action.

I've opened my demo app in Xcode. It's written in Swift and utilizes standard UIKit views. I've already written a few passing tests which verify that the elements on the screen exist.

```swift
import XCTest

final class WWDC23_SampleDemoUITests: XCTestCase {

	override func setUpWithError() throws {
		continueAfterFailure = false
	}

	func testQuoteTabView() throws {
		let app = XCUIApplication ()
		app.launch()
		app.tabBars.buttons ["Quote"].tap ()
	
		XCTAssert (app.images ["QUOTE_IMAGE"].exists)
		XCTAssert (app.textViews ["QUOTE_TEXTVIEW"].exists)
	}
	
	func testReflectTabView() throws {
		let app = XCUIApplication ()
		app.launch()
		app.tabBars.buttons["Reflect"].tap()
		
		XCTAssert(app.staticTexts["REFLECT_DATE_LABEL"].exists)
		XCTAssert(app.textViews["REFLECT_TEXT_VIEW"].exists)
}
```

![Accessibility inspector][tests]

[tests]: tests.jpg

For example, testQuoteTabView verifies that the image view and the text view exist in the quote tab. One thing to note is that these tests also help us test accessibility. In order for XCTest to find these views, they must be accessibility elements. That means if your UI tests can find the elements, so can our assistive technologies. 

I want to add some audits to my tests to make sure I'm catching all kinds of issues. I'll create another test called testAccessibilityQuoteTabView. 
I'll do some setup to launch my app and navigate to the Quote tab.
And finally, I'll call performAccessibilityAudit on the application.
```swift
func testAccessibilityQuoteTabView() throws {
	let app = XCUIApplication()
	app.launch()
	app.tabBars.buttons["Quote"].tap()
	
	try app.performAccessibilityAudit()
}
```

The audit can report multiple issues, so to allow my test to continue reporting issues after the first failure, I'll set continueAfterFailure to true.
```swift
	override func setUpWithError() throws {
		continueAfterFailure = true
	}
```
That's it. Let's run the test by clicking on the test diamond.  
Seems like my test failed. 

![Test Fail][testFail]

[testFail]: testFail.jpg

The issues are reported in-line within the Xcode source editor. My audit caught two issues: Element has no description, and the label is not human-readable. 

I can dig deeper into these two issues by going to the Report navigator, clicking on Tests, and then clicking on the disclosure triangle next to my test.

![Test Fail][testFail2]

[testFail2]: testFail2.jpg

For the first issue, "element has no description," I can double-click the element screenshot which shows me that the image view has no description.

![Test Fail][testFail3]

[testFail3]: testFail3.jpg

I can do so similarly for the second issue, which shows me that the label on the text view is not human-readable. 

![Test Fail][testFail4]

[testFail4]: testFail4.jpg

# Handling audit issues

To learn more about best practices in accessibility, please check out our talk:
[Deliver an Exceptional Accessibility Experience - WWDC18](https://developer.apple.com/videos/play/wwdc2018/230) 

## Fix issues individually
The accessibility label on the text view is not human-readable.

![Fixing the first issue][testFail5]

[testFail5]: testFail5.jpg

If I inspect the text view in the Storyboard, I can see that the accessibility label has been set to QUOTE_TEXTVIEW.

![Fixing the first issue][testFail6]

[testFail6]: testFail6.jpg

Users who rely on assistive technologies like VoiceOver will first hear the accessibility label, and then the accessibility value, like this. VoiceOver: QUOTE_TEXTVIEW, "Live one day at a time and make it a masterpiece.".  
The label doesn't sound great, and ideally, VoiceOver should skip it and speak just the quote itself.  

I can delete the accessibility label, but then my UI tests will break, because they depend on this label to identify the text view. Ideally, this string should be set as the accessibility identifier. The accessibility identifier allows you to uniquely identify an element when writing UI tests without affecting the accessibility or UI experience. I'll head over to my Storyboard.

I'll select my text view, cut this string from the label, and paste it into the identifier.

![Fixing the first issue][testFail7]

[testFail7]: testFail7.jpg

The other issue my audit found was that the image view has no description. In my app, this is a background image which is decorative. Ideally, technologies like VoiceOver should skip this image view.  
I can achieve this behavior by overriding accessibility elements on the view controller's view.
```swift
// Exclude the image view

view.accessibilityElements = [quoteTextView, newQuoteButton]
```

By setting it to just the quote text view and the New Quote button, VoiceOver will no longer land on the image view.  

Let's head over to Xcode and do that now. I'll go to my view controller file and set accessibilityElements.
```swift
import UIKit

class QuoteViewController: UIViewController {

	override func viewDidLoad () {
		super.viewDidLoad ()
		setupViews()
		
		// Exclude the image view
		view.accessibilityElements = [quoteTextView, newQuoteButton]
}
```

My audit is now passing. You'll notice one of my UI tests is now failing, but we'll come back to that later.  

## Filter issues out that might not be relevant
When adding accessibility audits, you may run into issues which need to be filtered.
As an example, let's say my audit found an issue with the contrast being too low on a specific label. The issue seems to be a false positive.  

The performAccessibilityAudit function takes in additional parameters. The first parameter allows me to specify an option set of the categories of audits that I want to run. These are categories like dynamic type and contrast, the same categories that you're already familiar with in the Accessibility Inspector.  
In this example, I'm choosing to audit for only dynamic type and contrast issues. The second parameter allows me to specify a closure. This closure is called on all the issues found by the audit and lets me choose which issues to ignore and which issues to report.  

I'll start by defining a variable called shouldIgnore to false.
```swift
// Ignore contrast issue on "My Label"

try app.performAccessibilityAudit(for: [.dynamicType, .contrast]) { issue in
		var shouldIgnore = false
		if let element = issue.element,
			element.label == "My Label",
			issue.auditType == .contrast {
				shouldIgnore = true
		}
	return shouldIgnore
}
```
![filtering issues to ignore because false positive][issues]

[issues]: issues.jpg

By default, issues should not be ignored. Let's say I'd like to ignore a contrast issue on an element with the label "My Label." I can get the XCUIElement associated with the issue using issue.element. If this element has the label "My Label" and the type of issue is a contrast issue, then I know I've got the right issue, so I'll set shouldIgnore to true.  
Setting it to true indicates that I'd like the issue to be ignored. At the end, I'll return shouldIgnore. If the conditions above aren't met, then shouldIgnore will be false, indicating the issue should be reported as a failure. And that's it. 

## Tips for Running audits
### Audit all views.
An audit is limited to the elements on the screen. 

### Override teardown
A quick way to immediately add audits for multiple tests is to override and perform the audit in teardown. You could define variables in the scope of the class. This way, tests can override these variables to opt in or out of the audit and to allow the tests to customize the closure for ignoring issues. 
### Use test plans
Test plans are an excellent way to group audit-specific tests in your project. They allow you to selectively enable test targets, cases, or individual methods in the test plan. 
### Test with assistive technologies
Ultimately, testing your app by turning on technologies like VoiceOver or Dynamic Type is the best way to ensure a high-quality experience. 


# Automation elements
Automation elements allow you to expose elements specifically for the purpose of automation without affecting the accessibility of those elements. Now, in UIKit, you'll be able to leverage this API to expose exactly the elements you need for automation, while still being able to customize the accessibility for these elements at the same time. 

You may remember from earlier that as I fixed the issues from my audit, I also broke one of my UI tests. The image view doesn't seem to be available anymore. It's missing in my UI test because it's also missing in accessibility. 

![it broke one of my UI tests. The image view doesn't seem to be available anymor][issues2]

[issues2]: issues2.jpg

Because this image view was decorative, I overrode accessibility elements to exclude it from accessibility. However, by doing so, I also caused it to become excluded from my UI test. Let's explore how automation elements can help me expose my image view to my UI test. 

I'll go to the view controller file in Xcode.
```swift
import UIKit

class QuoteViewController: UIViewController {

	override func viewDidLoad () {
		super.viewDidLoad ()
		setupViews()
		
		// Exclude the image view
		view.accessibilityElements = [quoteTextView, newQuoteButton]
		view.automationElements = [imageView, quoteTextView, newQuoteButton]
}
```

And I'll set automationElements on the view controller's view to the image view. When overriding automationElements, you need to specify all the elements which need to be exposed to automation.

That means I also need to add the text view and the button to my list. When overriding automation elements, you are overriding the existing elements that are already exposed to automation. 

# Wrapping up
We were able to write some great UI and accessibility tests and fix some accessibility issues too. Fixing the issues identified by the audits helps ensure everyone can enjoy your app. Create great accessibility and automation experiences without having to pick one over the other. Automation elements allows you to expose elements specifically for your UI tests without impacting the accessibility experience. 

# Resources
[Deliver an Exceptional Accessibility Experience - WWDC18](https://developer.apple.com/videos/play/wwdc2018/230)

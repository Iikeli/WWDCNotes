# Triage test failures with XCTIssue

Put your test failures to work: Learn how to triage and diagnose uncaught issues in your app using the latest testing APIs in Xcode. We’ll show you how to help ease your testing workflow and put failures into context to help you deliver the best quality product.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10687", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



## Overview
Maintaining any active project requires investigating test failures, either locally or in Continuous Integration (CI). Investigating test failures requires answering the following questions:

- What failed?
- How did it fail?
- Why did it fail?
- **Where** did it fail?

Xcode 12 makes answering these questions more efficient. This talk covers the following 4 topics:

1. Swift errors in tests
2. Rich failure objects
3. Failure call stacks
4. Advanced workflows

## Presenting Test Failures in Xcode 12
### Issue Navigator
This is an example of a test failure. Notice how the annotation is gray. This indicates that the failure happened underneath the indicated line. On the left, the issue navigator shows a call stack under the failed test.

![][gray_line]

Clicking on a frame takes the source editor to that location. Now, the annotation is red because this is the actual source of the error.

![][error_source]

### Test Report
The Test Report is another way to explore test failures, and is especially useful when working with the results of a CI failure.

Here, we can see a list of failing tests, each with a call stack. When you hover over a stack frame, you can see two buttons to the right. The Jump button, indicated with an arrow, jumps to the source of the error. The second button, called the Assistant button, is new in Xcode 12. This does the same thing as the Jump button, but opens a secondary editor next to the Test Report so you don't have to go back and forth. Clicking on a different stack frame changes the secondary editor immediately.

![][assistant_button]

## 1. Using Swift Errors in your tests
XCTest makes it possible for test functions to throw. When a test does throw, the error is used to formulate the error message.

This means you can replace boilerplate like this in your tests...

```swift
func testExample() {
  do {
    try codeThatThrows()
  } catch {
    XCTFail("\(error)")
  }
}
```

...with this:

```swift
func testExample() throws {
  try codeThatThrows()
}
```

However, until recently, these failures could not provide the source code location for thrown errors in tests. That has now been fixed with the Swift runtime improvements in the following platforms:

- iOS and tvOS 13.4
- macOS 10.15.4

### Throwing from setUp and tearDown
Similar improvements have also been brought to `setUp()` and `tearDown()`.

![][throwing_setup_teardown]

## 2. Rich Failure Objects
XCTest has always recorded test failures as 4 discrete values:

1. Failure message
2. File path
3. Line number where the failure was recorded
4. "Expected" flag to indicate if the failure was expected.

These values were passed into the `recordFailure` API, which ensures that failures are logged and routed to Xcode for display:

```swift
open class XCTestCase: XCTest {

  open func recordFailure(
    withDescription: String,
    inFile: String,
    atLine: Int,
    expected: Bool
  )

}
```

In Xcode 12, these values have been encapsulated into a new struct `XCTIssue`. This new type also adds:

- Distinct types
- Detailed description
- Associated error
- Attachments

### `XCTAttachment`
- Can capture arbitrary data
- Add this attachment to the test itself or an `XCTActivity`
- `XCTIssue` supports attachments, which allows customized diagnostics for test failures.

### Recording `XCTIssue`s
There is a new API on `XCTestCase` for recording test failures, which is called by all types of asserts. The old `recordFailure` API has been deprecated.

```swift
open class XCTestCase: XCTest {

  open func record(_ issue: XCTIssue)

}
```

### Modifying issues
In Swift, issues are immutable or mutable depending on your use of `let` or `var`:

```swift
let issue = XCTIssue(...)  // immutable

var issue = XCTIssue(...)  // mutable
```

In Objective-C, there is a special subclass for mutable issues which conforms to `NSMutableCopy`:

```objectivec
XCTIssue *issue = [[XCTIssue alloc] init ...];  // immutable

XCTMutableIssue *mutableIssue = [issue mutableCopy];  // mutable
```

## 3. Failures and Source Code
One of the most important questions to answer about a test failure is "where did it happen?".

### Before `XCTIssue`
Core test failure data always included file path and line number, which is great for simple tests. This is not enough for functions that are shared by more than one test.

Here are two tests calling the same shared function:

```swift
func testThingA() throws {
  let thing = Thing("a")
  try assertProperties(for: thing)
}

func testThingB() throws {
  let thing = Thing("b")
  try assertProperties(for: thing)
}

// Shared function
func assertProperties(for thing: Thing) throws {
  XCTAssertNotNil(thing.name)
  XCTAssertNotNil(thing.order)
}
```

Before, the test method would be marked as failing even though the failure occurred somewhere else. This has no further information to help the developer figure out **where** the failure happened.

![][shared_failure]

This can be mitigated if the helper functions captures information from where it was invoked and explicitly uses that information in its assertion calls. However, if the helper has more than 1 assertion, there is now a different ambiguity: which assertion failed in the helper?

![][capture_helper]

### Now with `XCTIssue`
**`XCTIssue` captures and symbolicates call stacks** so the point of failure is more clear. Here, the same code now has a gray annotation to demonstrate that the failure happens lower in the call stack, and a red annotation where the failure actually occurs.

![][failure_with_xctissue]

Most importantly, no extra effort was required to get this information.

## 4. Advanced Workflows with new APIs
### Advanced Workflow 1: Custom Assertions
You can make your own assertions by creating `XCTIssue` directly and then `record(_ issue:)`. In this example, the custom assertion validates some data and then includes the failure as an attachment to the issue that it records.

```swift
func assertSomething(
  about data: Data,
  file: StaticString = #filePath,
  line: UInt = #line
) {
  // Call out to custom validation function.
  guard !isValid(data) else { return }

  // Create issue, declare with `var` for mutability.
  var issue = XCTIssue(type: .assertionFailure, compactDescription: "Invalid data")

  // Attach the invalid data.
  issue.add(XCTAttachment(data: data))

  // Capture the call site location as the point of failure.
  let location = XCTSourceCodeLocation(filepath: file, lineNumber: line)
  issue.sourceCodeContext = XCTSourceCodeContext(location: location)

  // Record the issue.
  self.record(issue)
}
```

### Advanced Workflow 2: Overriding `record(_ issue:)`
You can observe, suppress, or modify failures recorded in your test class. This method is the funnel point through which all issues pass, so overrides have total control over the output of the test class.

#### Example 1: Overriding for Observation
```swift
override func record(_ issue: XCTIssue) {
  // Observe, introspect, log, etc:
  if shouldLog(issue) {
    print("I just observed an issue!")
  }

  // Don't forget to call super!
  super.record(issue)
}
```

You can also use `XCTestObservationCenter`, but this approach is more useful if you only want to observe failures in one class.

If your override does **not** call `super`, you will have suppressed the issue. It will not continue along the recording chain, and nothing will be sent to Xcode.

```swift
override func record(_ issue: XCTIssue) {
  // If you don't want to record it, just return.
  if shouldSuppress(issue) {
    return
  }

  // Otherwise, pass it to `super`.
  super.record(issue)
}
```

#### Example 2: Overriding for Modification
This is the most common reason for overriding because you can add attachments, which can be great diagnostic aids.

```swift
override func record(_ issue: XCTIssue) {
  // Redeclare using `var` to enable mutation.
  var issue = issue

  // Add a simple attachment.
  issue.add(XCTAttachment(string: "hello"))

  // Pass it to `super`.
  super.record(issue)
}
```

## Wrap Up
- Triaging test failures is important!
- Call stacks help answer "where?"
  - This helps support more natural factoring of test code, such as code reuse and other best practices
- Attachments improve "how?" and "why?"

[gray_line]: WWDC20-10687-gray_line

[error_source]: WWDC20-10687-error_source

[assistant_button]: WWDC20-10687-assistant_button

[throwing_setup_teardown]: WWDC20-10687-throwing_setup_teardown

[shared_failure]: WWDC20-10687-shared_failure

[capture_helper]: WWDC20-10687-capture_helper

[failure_with_xctissue]: WWDC20-10687-failure_with_xctissue

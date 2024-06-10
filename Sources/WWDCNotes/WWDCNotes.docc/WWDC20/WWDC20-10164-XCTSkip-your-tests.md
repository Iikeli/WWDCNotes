# XCTSkip your tests

Get the test results that matter — and skip the ones that don’t. Discover how you can implement XCTSkip to conditionally avoid tests at runtime. We'll take you through how to return this new test result and better document tests beyond pass and fail within your test bundle.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10164", purpose: link, label: "Watch Video (6 min)")

   @Contributors {
      @GitHubUser(abadikaka)
   }
}



## Conditional Test Execution

- Widely used in testing. 
- Determined on runtime. 

### What's new?

In Xcode 12 we can use [`XCTSkip`][skipDoc] for tests that require conditional test execution. It will give us one of the following result:

* Pass
* Fail
* Skip

### Examples

```swift
guard #available(iOS 13.4, *) else {
    throw XCTSkip("Pointer interaction tests can only run on iOS 13.4")
}
```

Our test will only run on iOS 13.4 or later: if the device running the test has a lower OS, then `XCTSkip` will be throw and the test is skipped.

We can also add conditions based on the device:

```swift
try XCTSkipIf(UIDevice.current.userInterfaceIdiom != .pad, "Pointer interaction tests are for iPad only")
```

Tests will be marked with gray color when skipped, and there will be an annotation to track the specified location skipped in the code.

![][test_1]

We're also able to check the results from the **test navigator** and **test report**. With the test report we can also see the reason why and where is the location of the skipped test

How does it look like in CI? See below:

![][test_2]

### APIs

There are 2 throwing function:

* [`XCTSkipIf`][skipIfDoc]: Skipa test if expression is `true`
* [`XCTSkipUnless`][skipUnlessDoc] : Skip test if expression is `false`

```swift
public func XCTSkipIf(
    _ expression: @autoclosure () throws -> Bool, 
    _ message: @autoclosure () -> String? = nil, 
    file: StaticString = #filePath, 
    line: UInt = #line
) throws

public func XCTSkipUnless(
    _ expression: @autoclosure () throws -> Bool, 
    _ message: @autoclosure () -> String? = nil, 
    file: StaticString = #filePath, 
    line: UInt = #line
) throws
```

Tests also can throw `XCTSkip` directly:

```swift
public struct XCTSkip: Error {

    public init(_ message: String? = nil, file: StaticString = #filePath, line: UInt = #line)

}
```

[test_1]: test_1.png
[test_2]: test_2.png

[skipDoc]: https://developer.apple.com/documentation/xctest/xctskip
[skipIfDoc]: https://developer.apple.com/documentation/xctest/3521325-xctskipif
[skipUnlessDoc]: https://developer.apple.com/documentation/xctest/3521326-xctskipunless

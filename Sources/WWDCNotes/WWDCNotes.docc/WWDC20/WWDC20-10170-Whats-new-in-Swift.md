# What's new in Swift

Join us for an update on Swift. Discover the latest advancements in runtime performance, along with improvements to the developer experience that make your code faster to read, edit, and debug. Find out how to take advantage of new language features like multiple trailing closures. Learn about new libraries available in the SDK, and explore the growing number of APIs available as Swift Packages.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10170", purpose: link, label: "Watch Video (32 min)")

   @Contributors {
      @GitHubUser(dasautoooo)
   }
}



## Runtime Performance
### Code Size
* Swift binary `__text` segment size vs. Objective-C binary `__text` segment size.
![][binarySizeImage]
* Swift needs some inevitable space to implement safety features.

### Dirty Memory
* Cut 2/3 of overhead's heap memory usage compared to Swift 5.1.
![][overheadCutImage]
* Lowering the Swift runtime in the userspace stack.

## Developer Experience

* In Apple systems, Swift's Standard Library has been moved below Foundation in the stack, making it available where previously C and Objective-C had to be used.

### Diagnostics
* Swift compiler has a new strategy for identifying code issues.
* More precise and actionable errors.
* Diagnostic Architecture [blogpost](https://swift.org/blog/new-diagnostic-arch-overview/) on swift.org.

### Code Completion
* Improved type-checking inference.
* KeyPath as function.

### Code Indentation
* Improved indentation formation.
  * Chained methods calls.
  * Call arguments.
  * Tuple elements.
  * Collection elements that span multiple lines.
  * Muti-line `if`, `guard` and `while` conditions.

### Debugging
* When debugging information is available, the debugger will display the reason for common Swift runtime failure traps.
* LLDB can resolve Objective-C types in Swift.

## Language
New language features:
![][languageFeaturesImage]

### [SE-0279](https://github.com/apple/swift-evolution/blob/master/proposals/0279-multiple-trailing-closures.md) Multiple trailing closure syntax
```swift
UIView.animate(withDuration: 0.3) {
	self.view.alpha = 0
} completion: { _ in
	self.view.removeFromSuperview()
}
```

#### API Design (Trailing closure syntax)
* Function name should clarify the role of first trailing closure, because the label will be dropped.

### [SE-0249](https://github.com/apple/swift-evolution/blob/master/proposals/0249-key-path-literal-function-expressions.md) KeyPath expressions as functions
* It's able to pass KeyPath arguments to any function parameters with matching signature.
* Introduces the ability to use the key path expression `\Root.value` wherever functions of `(Root) -> Value` are allowed.

```swift
extension Collection {
	func chunked<Key>(by keyForValue: @escaping (Element) -> Key) -> Chunked<Self, Key>
}

for (shoeSize, group) in contacts.chunked(by: \.shoeSize) {
	print(shoeSize)
	for contact in group {
		print("  ", contact.name)
	}
}
```

### [SE-0281](https://github.com/apple/swift-evolution/blob/master/proposals/0281-main-attribute.md) `@main`: Type-Based Program Entry Points
A Swift language feature for designating a type as the entry point for beginning program execution. Instead of writing top-level code, users can use the `@main` attribute on a single type. Libraries and frameworks can then provide custom entry-point behavior through protocols or class inheritance.

```swift
import ArgumentParser

@main
struct Hello: ParsableCommand {
	@Argument(help: "The name to greet.")
	var name: String
	
	func run() {
		print("Hello, \(name)!")
	}
}

```

### [SE-0269](https://github.com/apple/swift-evolution/blob/master/proposals/0269-implicit-self-explicit-capture.md) Increased availability of implicit self in closures
* Include `self` in the capture list, we can remove `self` in the body of closure.

```swift
UIView.animate(withDuration: 0.3) { [self] in
	recordingView?.alpha = 0.0
	textView.alpha =  1.0
} completion: { [self] _ in
	activeTransitionCount -= 1
	if !isRecording && activeTransitionCount == 0 {
		recordingView?.removeFromSuperview()
		recordingView = nil
	}
}
``` 

*  If `self` is a value type, it's no longer required to have any explicit usage of `self`. Such as `struct` or `enum`.

### [SE-0276](https://github.com/apple/swift-evolution/blob/master/proposals/0276-multi-pattern-catch-clauses.md) Multi-pattern catch clauses
```swift
do {
  try performTask()
} catch TaskError.someRecoverableError {    
  recover()
} catch TaskError.someFailure(let msg),
        TaskError.anotherFailure(let msg) {
  showMessage(msg)
}
```

### Enum enhancements
#### [SE-0266](https://github.com/apple/swift-evolution/blob/master/proposals/0266-synthesized-comparable-for-enumerations.md) Synthesized `Comparable` conformance for `enum` types
```swift
enum Membership: Int, Comparable {
    case premium
    case preferred
    case general

    static func < (lhs: Self, rhs: Self) -> Bool {
        return lhs.rawValue < rhs.rawValue
    }
}
```
#### [SE-0280](https://github.com/apple/swift-evolution/blob/master/proposals/0280-enum-cases-as-protocol-witnesses.md) Enum cases as protocol witnesses
```swift
let error1 = JSONDecodingError.fileCorruped
let error2 = JSONDecodingError.keyNotFound("shoeSize")

enum JSONDecodingError: DecodingError {
	case fileCorruped
	case keyNotFound(_ key: String)
}

protocol DecodingError {
	static var fileCorrupted: Self { get }
	static func keyNotFound(_ key: String) -> Self
}
```

### Embedded DSL enhancements
* Support for pattern matching control flow statement like `if let` and `switch`.
* More to watch session [10041](https://developer.apple.com/videos/play/wwdc2020/10041): What's new in SwiftUI.

## Libraries
### SDK
#### Float16
* IEEE 754 standard format. (Half-precision floating-point format)
  * Sign bit: 1 bit
  * Exponent width: 5 bits
  * Significand precision: 11 bits (10 explicitly stored)
  * Largest: 65504, Smallest: 0.000000059605
![][IEEE_754]

* Half-width floating point type takes 2 bytes(16 bits), single precision takes 4 bytes (32 bits).
* Low precision, small range.
* More to watch session [10217](https://developer.apple.com/videos/play/wwdc2020/10217): Explore numerical computing in Swift.

#### [Apple Archive](https://developer.apple.com/documentation/applearchive)
* Perform multithreaded lossless compression of directories, files, and data.
* Apple uses it to deliver OS updates.
* Modular archive format.
* Fast multithreaded compression.
* Command-line tool and Finder integration.

#### [OSLog](https://developer.apple.com/documentation/oslog)
* A unified logging system for the reading of historical data.

```swift
logger.log("\(offerID, align: .left(columns: 10), privacy: .public)")
// Logs "E1Z3F		"

logger.log("\(seconds, format: .fixed(precision: 2)) seconds")
// Logs "1.30 seconds"
```

* More to watch session [10168](https://developer.apple.com/videos/play/wwdc2020/10168): Explore logging in Swift.

### Packages
#### [Swift Numerics](https://github.com/apple/swift-numerics)
* Basic math functions for generic contexts.
* Complex numbers and arithmetic.
* Approximate equality.
* More to watch session [10217](https://developer.apple.com/videos/play/wwdc2020/10217): Explore numerical computing in Swift.

#### [Swift ArgumentParser](https://github.com/apple/swift-argument-parser)
* An open source package for command line argument parsing.
* Type-safe, declarative API.
* Synthesized help and error screens.
* Subcommand dispatch.
* Completion script generation.

#### [Swift StandardLibraryPreview](https://github.com/apple/swift-standard-library-preview) 
* Early access to new StandardLibary features.

[binarySizeImage]: WWDC20-10170-binary_size
[overheadCutImage]: WWDC20-10170-overhead_cut
[languageFeaturesImage]: WWDC20-10170-language_features
[IEEE_754]: WWDC20-10170-IEEE_754

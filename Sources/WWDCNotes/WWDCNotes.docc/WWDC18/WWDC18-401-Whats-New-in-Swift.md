# What's New in Swift

Hear about the latest advancements in Swift, the safe, fast, and expressive language. Find out about improvements to build times, code size, and runtime performance. Learn how to take advantage of new features in your code that eliminate boilerplate, increase safety and security, and improve your overall development productivity.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/401", purpose: link, label: "Watch Video (37 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- Faster builds
- Better performances for Swift tools
- Swift 5 is gonna be released in early 2019 (with ABI stability): 
  - Apps won’t need to be shipped with Swift embedded in the bundle because they can just use the iOS Swift runtime (after Swift 5 and a new iOS update is released)
  - This is a win for every app in memory usage and start up time
- With Xcode 10 we have two optimizations options: 
![][compilationImage]
  - Compilation Mode: this is how your project builds:
    - Whole Module: all the files in your target are always built together, this enables maximum opportunity for optimization (slow build but get best performance at run time)
    - Incremental: not all the files are always built all together (faster build but slower run time performance)

- 
  - ⚠️In debug mode you should use Incremental otherwise everytime you touch a file every single file in the whole project gets rebuilt ⚠️
  - Optimization Level: this is where you choose which optimization you’d like to have:
    - No optimization
    - Optimize for Speed: Faster code, bigger code output
    - Optimize for Size: Code size is reduced by 10/30%, performance is 5% slower

- (Swift’s) ARC improvements (which results in faster code runtime)

## New in Swift 4.2:

- Enums can conform to the new protocol `CaseIterable` to gain a new property `.allCases` (which returns an array of all the possible cases)

- Conditional Conformance: if you try to compare two collections (say two arrays, even with Optionals, Dictionaries) of elements that are comparable (aka Equatable), then the two arrays are automatically comparable as well. E.g.:

```swift
let coins = [[1, 2], [3, 6], [4, 12]]
coins.contains([3, 6]) // This now works!
```
 
- The same works for `Codable`, `Hashable` and `Decodable` conformances

- (Swift 4.1 actually) Synthesized `Equatable` and `Hashable` Conformance: A struct/class is automatically Equatable/Hashable if all its properties also are. This is super powerful because it works with generics as well! This:

```swift
enum Either<Left, Right> { 
  case left(Left)
  case right(Right) 
}

extension Either: Equatable where Left: Equatable, Right: Equatable {
  static func ==(a: Either<Left, Right>, b: Either<Left, Right>) { 
    switch (a, b) { 
      case (.left (let x), .left (let y)): return X == y 
      case (.right (let x), .right(let y)): return x == y
      default: return false
    }
  }
} 
```

Is now this:

```swift
enum Either<Left, Right> { 
  case left(Left)
  case right(Right) 
}

extension Either: Equatable where Left: Equatable, Right: Equatable { }
extension Either: Hashable where Left: Hashable, Right: Hashable { }
```

- Hashable Improvements: The `Hashable` protocol doesn’t require a hash `Int` property anymore but:

```swift
protocol Hashable {
  func hash(into hasher: inout Hasher)
}
```

You can feed the Hasher with multiple values to hash together and the Hasher will take care of creating the hash for you. This is how you now conform to `Hashable`:

```swift
extension City: Hashable {
  func hash(into hasher: inout Hasher) {
    name.hash(into: &hasher)
    state.hash(into: &hasher)
  }
}
```

⚠️Hash values will change every time you run the app.
 
- Generating Random Numbers:
  - All numeric types have a static random function: the new method, which takes a range as a parameter, returns a random number within that range.
  - Bool also get a `.random()`
  - Collections get a `.randomElement()` method, which returns a `Element?` (optional because if the collection is empty this method returns `nil`)
  - Arrays get a `.shuffled()` method that returns the same array with elements put in a random order 😍
  - All the methods above have an optional random number generator parameter input that you can use if you want to use your own random number generator instead of the default Swift one.

- Instead of using:

```swift
#if os(iOS) || os(watchOS) || os(tvOS)
  import UIKit
  ...
#else
  import AppKit
  ...
#endif
```

You can now use:

```swift
#if canImport(UIKit)
  import UIKit
  ...
#elseif canImport(AppKit)
  import AppKit
  ...
#else
  #error("Unsupported platform")
#endif
```

> note the `canImport` macro and the new build error `#error`.

- No more FIXME/TODOs in Swift, use #warning instead:

```swift
#warning("We need to test this better")
```

- This session completely ignores `@dynamicMemberLookup`.

[compilationImage]: WWDC18-401-compilation
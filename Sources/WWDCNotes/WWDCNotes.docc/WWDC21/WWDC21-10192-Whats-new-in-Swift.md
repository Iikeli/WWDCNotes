# What‘s new in Swift

Join us for an update on Swift. Discover the latest language advancements that make your code easier to read and write. Explore the growing number of APIs available as Swift packages. And we’ll introduce you to Swift’s async/await syntax, structured concurrency, and actors.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10192", purpose: link, label: "Watch Video (32 min)")

   @Contributors {
      @GitHubUser(rayfix)
   }
}



## Overview

Swift 5.5 is the best release yet, including:

- Async and Concurrent programming
- Advances in the packages experience
- New standard library packages
- Features enhancing dev experience

## Diversity

❤️ The heart of the Swift Project is not the code but the community.

Diversity is a core value. 

Evidence shows open source projects that have a diverse community with a wide set
of perspectives helps the community thrive and the project make better decisions.

[Swift.org :: Diversity in Swift](https://swift.org/diversity/)

"The mission of Diversity in Swift is to foster an inclusive Swift community by 
creating more pathways for a diverse group of developers, increasing the engagement and 
retention of those developers, and helping developers of all backgrounds establish 
leadership and technical expertise within the community."

Highlight a variety of voices in the community. Community groups to help those with a
similar background get started.

## 📦 Swift Packages

Being able to find and use open source packages is an important part of the ecosystem.
The community built [Swift Package Index](https://swiftpackageindex.com) to help with this. 🎉🎉🎉
**Aside**: Thank you [Dave Verwer, Sven A. Schmidt. and others](https://github.com/SwiftPackageIndex/SwiftPackageIndex-Server/graphs/contributors).

In Xcode 13 and Swift 5.5, you can find packaged directly.

## 📦📦📦 Swift Package Collections

- Search screen in Xcode
- Anyone can publish them (JSON file)
- Curated list of of packages for different use cases
- Searchable

Example: a set of packages for a computer science class, a set for a specific problem domain, organization, etc.

Xcode comes pre-wired with a set of Apple standard packages.  
See [swift.org/blog/package-collections](https://swift.org/blog/package-collections) for more details.

### Apple Packages

- Swift Algorithms **(new)**
- Swift Crypto
- Swift Argument Parser
- Swift Atomics **(new)**
- Swift Collections **(new)**
- Swift Protobuf
- Swift NIO
- Swift Numerics
- Swift System **(new)**

## ᠅ [Swift Collections](https://github.com/apple/swift-collections)

Similar to Swift standard library types. 

- Deque (pronounced deck)
- Ordered Set
- Ordered Dictionary

Deque has a fast O(1) append and prepend operation

Ordered set is like an array in that it maintains order and random access but like a set because it makes sure elements are unique (and has fast lookup).

Ordered dictionary is an alternate to dictionary when order is important or you need random access to elements.

## Swift Algorithms

Open source package of algorithms on sequence and collection types.  There are already over 40 algorithms there.

- generating all the combinations or permutations
- iterating by groups (chunks)
- selecting smallest / largest / random elements from a collection

There is a session about algorithms and collections and how you can use them to make your code better.

## [Swift System](https://github.com/apple/swift-system/)

Idiomatic, low-level interfaces to system calls.

- Strong types
- Memory safety
- Error handling
- macOS, Linux and Windows support

Supports thing like `FileDescriptors`, `FilePath`.

```swift
import System

var path: FilePath = "/tmp/WWDC2021.txt"
print(path.lastComponent)
print(path.extension)
path.extension = nil
path.extension = "pdf"
// etc
```

## [Swift Numerics](https://github.com/apple/swift-numerics)

New this year

- `Float16` and `Complex<Float>`
- Elementary functions support for `Complex`
- Optimizations above the C library versions.

## [ArgumentParser](https://github.com/apple/swift-argument-parser)

Improvements this year:

- Code completion for Fish shell
- Joined short options (`-Ddebug`)
- Improved error messages

Now used by Swift Package Manager in Xcode 12.5.

## Swift on Server

- Static linking on Linux
- Improved JSON performance
- Enhanced AWS Lambda runtime library

Cold start 33% faster

40% faster invocation times on AWS gateway

Uses new async/await instead of completion closures

## DocC

Documentation compiler integrated in Xcode 13

- Markdown in source code

There are four other sessions about this

- It will become open source later this year.  🥳🥳🥳

## Improvements to Type Checker

- Fewer expression too complex error
- Array literals type checking sped up

## Build Improvements

- Faster incremental builds when changing imported modules
- Faster startup time before launching compiles
- Fewer recompilations after changing extension body.

As an example, now less than a tenth of the files rebuilt after a module changes resulting in a one-third speedup.  Incremental builds.

The first part of the compiler (the driver) is now written in Swift.  It is now the default.

## Memory Management

Class instances use ARC (automatic reference counting).

The compiler now uses a smarter way to track references and can eliminate retain release traffic. 

Improves runtime and code size but is an compiler option "Optimize Object Lifetimes"

There is a session about this.

## Ergonomic improvements

- [SE-0284][SE-0284]: Multiple variadic parameters
- [SE-0287][SE-0287]: Implicit member chains
- [SE-0289][SE-0289]: Result builders
- [SE-0293][SE-0293]: Property wrappers on parameters
- [SE-0295][SE-0295]: Codable synthesis for associated value enums
- [SE-0299][SE-0299]: Static member lookup in generic contexts
- [SE-0307][SE-0307]: Interchangeable use of CGFloat and Double
- [SE-0308][SE-0308]: #if for postfix member expressions

The SE standards for Swift Evolution.

What follows is a series of examples of how these features can simplify your code.

- Result builders was refined over the year and now has a whole session devoted to it.
- Making enums with associated values `Codable`, instead of being pages of boilerplate is now just adding `Codable` and letting the compiler do the work.

## Flexible static member lookup

```swift
protocol Coffee { ... }
struct RegularCoffee: Coffee {}
struct Cappuccino: Coffee {}

extension Coffee where Self == Cappucino {
  static var cappucino: Cappucino = { Cappucino() }
}

func brew<CoffeeType: Coffee>(_: CoffeeType) { ... }

brew(.cappucino.large)   // Beautify enum-like syntax!
```

### Improved property wrappers

You can use property wrappers can now be used on function (and closures) parameters.

## SwiftUI example

The before and after is pretty remarkable!

In Swift you can pass a binding to SwiftUI `List` to get access to the projection and wrapped value.

There is a session about this.

It makes your code simpler!

## Asynchronous and Concurrent Programming

- Synchronous: code statements execute one-by-one and in-order
- Asynchronous: code suspends execution waiting for op to finish, then continues
- Concurrent: multiple code statements executing at once

Without Swift's new features you often write async code using completion handlers (closure).

Later, after the network request finishes, the callback happens.
Only then can you deal with errors, which can be awkward.

`URLSession`'s `dataTask` is changed to this:

```swift
let (data, response) = try await URLSession.shared.data(for: request)
```

The await will suspend the operation but **not** block the thread so other 
work can continue on that thread.  This allows a small number of threads 
to be shared among several asynchronous processes.

The code can then continue inline making it easy to follow.

There are other sessions about this.

## Concurrency

Concurrency builds on the async/await idea.

```swift
func titleImage() async throws -> Image {
  async let background = renderBackground()
  async let foreground = renderForeground()
  let title = try renderTitle()
  return try await merge(background, foreground, title)
}
```

The `async let` spawns off another concurrent operation.  
The `merge` operation has to be marked with an `await` to signal that
this thread may suspend.  This function **will not return** until the
background tasks complete.  Even if an error is thrown, Swift 
will ensure everything completes before returning.

Swift will signal unfinished tasks that an error was thrown as an optimization.
The Structured concurrency in Swift session describes this in greater detail.

## Actors

This is another concurrency construct that helps you manage shared mutable data.  Here is an example
that will corrupt if accessed from multiple threads:

```swift
class Statistics {
  private var counter: Int = 0
  func increment() {
    counter += 1
  }
}
```

The fix is simple.  Change `class` to `actor`:

```swift
actor Statistics {
  private var counter: Int = 0
  func increment() {
    counter += 1
  }
}
```

The Swift compiler prevents corruption by suspending operations 
until it is safe to make the change.  That is why you typically
need to call them with `await`.

They also work with async/await.

Actors are like classes (reference types) but allow the Swift compiler 
and runtime to protect against data races.

There is a full session about this.

## Future of Swift 6

- async/await/actors is a basis
- Swift 6 will have additional capability enable "Safe concurrency"
- Make concurrent programming just as hard as normal programming
- More efficiency 

Make Swift 6 better by telling swift.org about your experiences!

You can try a new compiler toolchain!

Participate in the forums

Participate in the mentorship program. 
The only requirement is a desire to improve Swift for everyone in the community.

[SE-0284]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0284-multiple-variadic-parameters.md
[SE-0287]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0287-implicit-member-chains.md
[SE-0289]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0289-result-builders.md
[SE-0293]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0293-extend-property-wrappers-to-function-and-closure-parameters.md
[SE-0295]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0295-codable-synthesis-for-enums-with-associated-values.md
[SE-0299]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0299-extend-generic-static-member-lookup.md
[SE-0307]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0307-allow-interchangeable-use-of-double-cgfloat-types.md
[SE-0308]: https://github.com/apple/swift-evolution/blob/be686706919cab796b47b8c528a1fa52028a8335/proposals/0309-unlock-existential-types-for-all-protocols.md
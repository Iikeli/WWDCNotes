# Protect mutable state with Swift actors

Data races occur when two separate threads concurrently access the same mutable state. They are trivial to construct, but are notoriously hard to debug.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10133", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Data races occur when:

- Two (or more) threads concurrently access the same data
- One of them is a write

Shared mutable state in concurrent programs:

- Shared mutable state requires synchronization - this synchronization ensures that concurrent use of our shared mutable state won't cause data races
- various primitives exist: atomics, locks, serial dispatch queues

## Actors

- Actors provide synchronization for shared mutable state
- Actors isolate their state from the rest of the program
  - the only way to access that state is by going through the actor

```swift
actor Counter { // 👈🏻
  var value = 0

  /// When this method is called, it is guaranteed by the actor that it will 
  /// run to completion without any other code executing on the actor.
  func increment() -> Int {
    value = value + 1
    return value
  }
}

let counter = Counter()

Task.detached {
  //     👇🏻 Whenever you interact with an actor from the outside, you do so asynchronously
  print(await counter.increment())
}

Task.detached {
  //     👇🏻 Whenever you interact with an actor from the outside, you do so asynchronously
  print(await counter.increment())
}
```

> Two calls to `counter.increment()` will always bring to the same result, it is guaranteed that we will never encounter an [Readers–writers problem][Readers–writers problem].

Actors provide the same capabilities as all of the named types in Swift:

- They can have properties, methods, initializers, subscripts, and so on
- They can conform to protocols and be augmented with extensions
- They are reference types; because the purpose of actors is to express shared mutable state

The primary distinguishing characteristic of actor types is that they isolate their instance data from the rest of the program and ensure synchronized access to that data. Actor eliminates the potential for data races on the actor's state.

Note that:

- Calls within an actor are synchronous
- Synchronous code always runs uninterrupted

```swift
extension Counter {
  func resetSlowly(to newValue: Int) {
    value = 0
    for _ in 0..<newValue {
      increment() // no need await, as we're already running code within the actor
    }
    assert(value == newValue)
  }
}
```

## Actor reentrancy

We are building an image downloader actor:

```swift
actor ImageDownloader {
  private var cache: [URL: Image] = [:]

  func image(from url: URL) async throws -> Image? {
    if let cached = cache[url] { return cached }

    let image = try await downloadImage(from: url)
    cache[url] = image // 👈🏻 Potential bug: `cache` may have changed
    return image
  }
}
```

Despite running on an actor, we have a bug:

1. imagine triggering `image(from:)`, missing cache and await on `downloadImage(from:)`, at that point the execution suspends
2. while waiting we trigger `image(from:)` again, with the same url
3. because the first run is suspended, the second will run until it awaits as well on `downloadImage(from:)`
4. at some point in the future both calls will return and will write to the same `cache[url]` place

> Remember: actor guarantees that only one flow can run within the actor at any given time, but when an execution suspends, others can run on the same actor.

The potential bug is on the fact that the backend might have changed image data between the two `downloadImage(from:)` calls, making our app return different images between the different calls.

In this case, the fix is to replace the image cache only if it is still missing from the cache after the `downloadImage(from:)` call:

```swift
actor ImageDownloader {
  private var cache: [URL: Image] = [:]

  func image(from url: URL) async throws -> Image? {
    if let cached = cache[url] {
      return cached
    }

    let image = try await downloadImage(from: url)

    cache[url] = cache[url, default: image] // 👈🏻
    return cache[url]
  }
}
```

A better solution would be to avoid downloading the same image multiple times:

```swift
actor ImageDownloader {

  private enum CacheEntry {
    case inProgress(Task<Image, Error>)
    case ready(Image)
  }

  private var cache: [URL: CacheEntry] = [:]

  func image(from url: URL) async throws -> Image? {
    if let cached = cache[url] {
      switch cached {
        case .ready(let image):
          return image
        case .inProgress(let task):
          return try await task.value
      }
    }

    let task = Task {
      try await downloadImage(from: url)
    }

    cache[url] = .inProgress(task)

    do {
      let image = try await task.value
      cache[url] = .ready(image)
      return image
    } catch {
      cache[url] = nil
      throw error
    }
  }
}
```

Actor reentrancy prevents deadlocks and guarantees forward progress, but it requires you to check your assumptions across each await.

Reentrancy tips:

- Perform mutation of actor state within synchronous code. Ideally, do it within a synchronous function so all state changes are well-encapsulated
- State changes can involve temporarily putting our actor into an inconsistent state - make sure to restore consistency before an `await`
- Expect that the actor state could change during suspension - all `await`s are potential suspension points
- Check your assumptions after resuming (after an `await`)

## Actor isolation

### Extensions

Consider the following actor definition and extension:

```swift
actor LibraryAccount {
  let idNumber: Int
  var booksOnLoan: [Book] =[]
}

extension LibraryAccount: Equatable {
  static func ==(lhs: LibraryAccount, rhs: LibraryAccount) -> Bool {
    lhs.idNumber == rhs.idNumber
  }
}
```

The static equality method compares two library accounts based on their ID numbers. Because the method is static, there is no `self` instance and so it is not isolated to the actor. Instead, we have two parameters of actor type, and this static method is outside of both of them. That's OK because the implementation is only accessing immutable state (`let idNumber`) on the actors.

Consider the following extension:

```swift
extension LibraryAccount: Hashable {
  func hash(into hasher: inout Hasher) {
    hasher.combine(idNumber)
  }
}
```

This time it is not ok: conforming to `Hashable` this way means that `hash(into:)` could be called from outside the actor, and this method is not `async`, so there is no way to maintain actor isolation. To fix this, we can make this method `nonisolated`:

```swift
extension LibraryAccount: Hashable {
  // 👇🏻
  nonisolated func hash(into hasher: inout Hasher) {
    hasher.combine(idNumber)
  }
}
```

- `nonisolated` means that this method is treated as being outside the actor, even though it is, syntactically, described on the actor
- This means that it can satisfy the synchronous requirement from the `Hashable` protocol
- Because `nonisolated` methods are treated as being outside the actor, they cannot reference mutable state on the actor

### Closures

- Like functions, a closure might be actor-isolated or it might be nonisolated

```swift
extension LibraryAccount {
  func readSome(_ book: Book) -> Int { ... }
  
  func read() -> Int {
    booksOnLoan.reduce(0) { book in
      readSome(book)
    }
  }
}
```

## Sendable types

- Sendable types are types whose values can be safely shared across different actors:
  - value types - because each copy is independent
  - Actor types - because they synchronize access to their mutable state
  - Immutable classes - as they're read-only
  - Internally-synchronized classes - for example with a lock
  - `@Sendable` function types
- all of your concurrent code should primarily communicate in terms of `Sendable` types
- `Sendable` types protect code from data races

### Conforming to `Sendable`

- [`Sendable` is a protocol][sendable]
- Swift will then check to make sure your type makes sense as a `Sendable` type

```swift
struct Book: Sendable {
  var title: String
  var authors: [Author] // Author is a struct type
}
```

- This struct can be `Sendable`, as all of its stored properties are of `Sendable` type
- If `Author` was a non-`Sendable` class, then this code would not compile

For generic types, we can use conditional conformance to propagate `Sendable` when it's appropriate:

```swift
struct Pair<T, U> {
  var first: T
  var second: U
}

extension Pair: Sendable where T: Sendable, U: Sendable {}
```

`@Sendable` functions:

- `@Sendable` function types conform to the `Sendable` protocol
- `@Sendable` places restrictions on closures:
  - No mutable captures - otherwise it'd allow data races on the local variable
  - Captures must be of `Sendable` type - this makes sure that the closure cannot be used to move non-Sendable types across actor boundaries
  - Cannot be both synchronous and actor-isolated - otherwise it'd allow code to be run on the actor from the outside

- `Task.detached(operation:)` accepts a `@Sendable` closure:

```swift
static func detached(operation: @Sendable () async -> Success) -> Task<Success, Never>
```

## Main actor

- special actor that represents the main thread
- differs from a normal actor in two ways:
  1. the main actor performs all of its synchronization through the main dispatch queue - from a runtime perspective, the main actor is interchangeable with using DispatchQueue.main
  2. the code and data that needs to be on the main thread is scattered everywhere

```swift
@MainActor func checkedOut(_ booksOnLoan: [Book]) {
  booksView.checkedOutBooks = booksOnLoan
}

// Swift ensures that this code is always run on the main thread.
await checkedOut(booksOnLoan)
```

- By marking code that must run on the main thread as being on the main actor, there is no more guesswork about when to use `DispatchQueue.main`
- Swift ensures that this code is always executed on the main thread

Types can be placed on the main actor:

- Implies that all methods and properties of the type are `MainActor`
- Opt out individual members with `nonisolated`

```swift
@MainActor class MyViewController: UIViewController {
  func onPress(...) { ... } // implicitly @MainActor

  nonisolated func fetchLatestAndDisplay() async { ... } 
}
```

[sendable]: https://developer.apple.com/documentation/swift/sendable
[Readers–writers problem]: https://en.wikipedia.org/wiki/Readers–writers_problem
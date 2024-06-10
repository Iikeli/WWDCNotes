# Meet Swift Async Algorithms

Discover the latest open source Swift package from Apple: Swift Async Algorithms. We'll explore algorithms from this package that you can use with AsyncSequence, including zip, merge, and throttle. Follow along with us as we use these algorithms to build a great messaging app. We'll also share best practices for combining multiple AsyncSequences and using the Swift Clock type to work with values over time.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110355", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## [Swift Async Algorithms][swift-async-algorithms] open-source package

- set of algorithms specifically focused on processing values over time using [`AsyncSequence`][asyncsequence]
- incorporates more advanced algorithms, as well as interoperating with clocks

## [`AsyncSequence`][asyncsequence]

- protocol that lets you describe values produced asynchronously
- just like `Sequence` but uses Swift concurrency (`next()` function from its iterator, `AsyncIterator`, is `async`)

### Multi-input algorithms

- algorithms that take multiple input `AsyncSequences` and produce one output `AsyncSequence`
- focused on combining `AsyncSequences` together in different ways

Examples: 

- [`zip(_:...)`][zip(_:...)] - creates an asynchronous sequence of pairs built out of underlying asynchronous sequences

```swift
for try await (vid, preview) in zip(videos, previews) {
  try await upload(vid, preview)
}
```

- [`merge(_:...)`][merge(_:...)] - merges two or more asynchronous sequence into a single asynchronous sequence producing the elements of all of the underlying asynchronous sequences (elements types must be the same)

```swift
for try await message in merge(primaryAccount.messages, secondaryAccount.messages) {
  displayPreview(message)
}
```

## Clock, Instant, and Duration 

- new API in Swift 5.7

### `Clock`

- Swift protocol, defines:
  - a way to wake up after a given instant
  - a way to produce a concept of "now"

Two built-in `Clock`s definitions are `ContinuousClock` and the `SuspendingClock`:

- use `ContinuousClock` to measure time just like a stopwatch, where time progresses no matter the state of the thing being measured
- `SuspendingClock`, on the other hand, suspends when the machine is put to sleep


```swift
// Sleep until a given deadline

let clock = SuspendingClock() 
var deadline = clock.now + .seconds(3)
try await clock.sleep(until: deadline) 
```

The key difference between these two clocks is that the `ContinuousClock` progresses while/when/if the machine is asleep.

Use `SuspendingClock` for:

- Measuring device time
- Delays for animation

Use `ContinuousClock` for:

- Measuring human time
- Delays by an absolute duration

### Swift Async Algorithms

- the package brings in a family of algorithms to work with time by leveraging these new Swift APIs

Examples:

- [`debounce(for:tolerance:clock:)`][debounce(for:tolerance:clock:)]

```swift
let queries = searchValues.debounce(for: .milliseconds(300))

for await query in queries {
  let results = try await performSearch(query)
  await channel.send(results)
}
```

- [`chunks(...)` and `chunked(...)`][chuck] - groups elements into collections by count/time/content

```swift
let batches = outboundMessages.chunked(
  by: .repeating(every: .milliseconds(500))
)

let encoder = JSONEncoder() 
for await batch in batches {
  let data = try encoder.encode(batch)
  try await postToServer(data) 
}
```

## Collections

- the package offers a set of initializers for constructing collections using `AsyncSequence` (array, set, dictionary)

[swift-async-algorithms]: https://github.com/apple/swift-async-algorithms
[asyncsequence]: https://developer.apple.com/documentation/swift/asyncsequence
[zip(_:...)]: https://github.com/apple/swift-async-algorithms/blob/main/Sources/AsyncAlgorithms/AsyncAlgorithms.docc/Guides/Zip.md
[merge(_:...)]: https://github.com/apple/swift-async-algorithms/blob/main/Sources/AsyncAlgorithms/AsyncAlgorithms.docc/Guides/Merge.md
[debounce(for:tolerance:clock:)]: https://github.com/apple/swift-async-algorithms/blob/main/Sources/AsyncAlgorithms/AsyncAlgorithms.docc/Guides/Debounce.md
[chuck]: https://github.com/apple/swift-async-algorithms/blob/main/Sources/AsyncAlgorithms/AsyncAlgorithms.docc/Guides/Chunked.md

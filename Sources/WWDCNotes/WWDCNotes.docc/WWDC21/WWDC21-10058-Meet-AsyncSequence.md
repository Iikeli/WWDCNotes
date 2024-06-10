# Meet AsyncSequence

Iterating over a sequence of values over time is now as easy as writing a “for” loop. Find out how the new AsyncSequence protocol enables a natural, simple syntax for iterating over anything from notifications to bytes being streamed from a server. We'll also show you how to adapt existing code to provide asynchronous sequences of your own.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10058", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(donnywals)
      @GitHubUser(zntfdr)
   }
}



Map, filter, reduce, dropFirst all work in async sequences, for example:

```swift
/// This tool prints lines as we receive the `all_month.csv` file, 
/// we don't need to wait for the complete file to download first.
@main
struct QuakesTool {
  static func main() async throws {
    let endpointURL = URL(string: "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.csv")!

    // Skip the header line and iterate each one to extract the magnitude, time, latitude and longitude
    for try await event in endpointURL.lines.dropFirst() {
      let values = event.split(separator: ",")
      let time = values[0]
      let latitude = values[1]
      let longitude = values[2]
      let magnitude = values[4]
      print("Magnitude \(magnitude) on \(time) at \(latitude) \(longitude)")
    }
  }
}
```

Async/await key points:

- Async functions let you write concurrent code without the need for callbacks, by using the `await` keyword
- calling an async function will suspend the current flow, which is then resumed whenever a value or error is produced

## What is AsyncSequence

- AsyncSequence will suspend on each element and resume when the underlying iterator produces a value or throws
- just like other sequences, except async - each element is delivered asynchronously
- may throw
- AsyncSequences either completes with success or stops when an error is thrown - depending if failure is an option
- An async iterator also consumes its underlying collection.

Generally speaking, asynchronous sequences are a description of how to produce values over time:

- an async sequence may be zero or more values and then signify completion with returning a nil from its iterator, just like sequences
- When an error occurs, the async sequence is at a terminal state - after an error happens, they'll return `nil` for any subsequent calls to `next` on their iterator

Things like `break` and `continue` work in async sequences too.

You can cancel an iteration by holding on to its `Task.Handle` when you wrap it in `async`:

```swift
let handle = async {
  for await thing in list {
    // ...
  }
}

handle.cancel()
```

## Compiler transformation

Take the following code:

```swift
for quake in quakes {
  if quake.magnitude > 3 {
    displaySignificantEarthquake(quake)
  }
}
```

The compiler will transform it into:

```swift
var iterator = quakes.makeIterator()
while let quake = iterator.next() {
  if quake.magnitude > 3 {
    displaySignificantEarthquake(quake)
  }
}
```

if `quakes` was an asyncSequence, it'd turn into:

```swift
var iterator = quakes.makeAsyncIterator()
while let quake = await iterator.next() {
  if quake.magnitude > 3 {
    displaySignificantEarthquake(quake)
  }
}
```

## API highlight

Read bytes asynchronously from a `FileHandle`:

```swift
// bytes: AsyncBytes
for try await line in FileHandle.standardInput.bytes.lines {
  ...
}
```

Read lines asynchronously from a `URL`:

```swift
// lines: AsyncLineSequence<AsyncBytes>
let url = URL(fileURLWithPath: "/tmp/somefile.txt")
for try await line in url.lines {
  ...
}
```

Read bytes asynchronously from a `URLSession`:

```swift
// you can use:
// func bytes(from: URL) async throws -> (AsyncBytes, URLResponse)
// func bytes(for: URLRequest) async throws -> (AsyncBytes, URLResponse)

let (bytes, response) = try await URLSession.shared.bytes(from: url)

guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 /* OK */
else { throw MyNetworkingError.invalidServerResponse }

for try await byte in bytes {
  ...
}
```

Await notifications asynchronously:

```swift
let center = NotificationCenter.default
let notification = await center.notifications(named: .NSPersistentStoreRemoteChange).first {
  $0.userInfo[NSStoreUUIDKey] == storeUUID
}
```

## How to create async sequences

Pretty much anything that does not need a response back and is just informing of a new value that occurs can be a prime candidate for making an async sequence.

Definition:

```swift
public struct AsyncStream<Element>: AsyncSequence {
  public init(
    elementType: Element.Type = Element.self,
    maxBufferedElements limit: Int = .max,
    _ build: (Continuation) -> Void
  )
}

// If we need to handle errors:
public struct AsynchrowingStream<Element>: AsyncSequence {
  public init(
    elementType: Element.Type = Element.self,
    maxBufferedElements limit: Int = .max,
    _ build: (Continuation) -> Void
  )
}
```
Example:

```swift
class QuakeMonitor {
  var quakeHandler: (Quake) -> Void 
  func startMonitoring()
  func stopMonitoring()
}

let quakes = AsyncStream(Quake.self) { continuation in
  let monitor = QuakeMonitor()
  monitor.quakeHandler = { quake in
    continuation.yield(quake)
  }
  continuation.onTermination = { @Sendable _ in
    monitor.stopMonitoring()
  }
  monitor.startMonitoring()
}

let significantQuakes = quakes.filter { quake in
  quake.magnitude > 3
}

for await quake in significantQuakes {
  ...
}
```

- when constructing an async stream, an element type (`Quake`) and construction closure is specified
- The closure has a `continuation` that can yield values more than once, finish, or handle termination

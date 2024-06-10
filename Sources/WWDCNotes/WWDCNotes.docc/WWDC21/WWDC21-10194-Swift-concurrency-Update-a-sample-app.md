# Swift concurrency: Update a sample app

Discover Swift concurrency in action: Follow along as we update an existing sample app. Get real-world experience with async/await, actors, and continuations. We’ll also explore techniques for migrating existing code to Swift concurrency over time.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10194", purpose: link, label: "Watch Video (61 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Adopting new async frameworks APIs:

- use `Task { await insertAwaitFuncHere }` in your synchronous code to spin off a new asynchronous task which will be allowed to call async functions

Create async APIs:

- while refactoring to async, create an async version of your completion-based APIs
  - Xcode can do this for you by using the <kbd>Create Async Alternative</kbd> refactoring action on the legacy API

- Use continuations when you must use a completion-block based API in your async code:

```swift
private func queryHealthKit() async throws -> ([HKSample]?, [HKDeletedObject]?, HKQueryAnchor?) {
  return try await withCheckedThrowingContinuation { continuation in // 👈🏻
    // Create a predicate that only returns samples created within the last 24 hours.
    let endDate = Date()
    let startDate = endDate.addingTimeInterval(-24.0 * 60.0 * 60.0)
    let datePredicate = HKQuery.predicateForSamples(withStart: startDate, end: endDate, options: [.strictStartDate, .strictEndDate])

    // Create the query.
    let query = HKAnchoredObjectQuery(
      type: caffeineType,
      predicate: datePredicate,
      anchor: anchor,
      limit: HKObjectQueryNoLimit) { (_, samples, deletedSamples, newAnchor, error) in

      // When the query ends, check for errors.
      if let error = error {
        continuation.resume(throwing: error) // 👈🏻
      } else {
        continuation.resume(returning: (samples, deletedSamples, newAnchor)) // 👈🏻
      }
    }
    store.execute(query)
  }
}
```

Use `@MainActor` to coordinate operations on the main thread, two ways:

- call `await MainActor.run { ... }` - this takes a block of code to run on the main actor
- annotate functions with `@MainActor` - to require that the caller switch to the main actor before this function is run

Turn your classes into `actors` or `@MainActor class` to avoid data races 

- Unlike the main actor, which is a global actor, other actor type can be instantiated multiple times
- use the `nonisolated` attribute in actor functions that are not going to touch any part of the actor isolated state - therefore, these functions can be called from anywhere

```swift
@available(*, deprecated, message: "Prefer async alternative instead")
nonisolated public func requestAuthorization(completionHandler: @escaping (Bool) -> Void ) {
  // ...
}

@available(*, deprecated, message: "Prefer async alternative instead")
nonisolated public func loadNewDataFromHealthKit(completionHandler: @escaping (Bool) -> Void = { _ in }) {
  // ...
}
```
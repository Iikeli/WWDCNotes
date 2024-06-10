# Eliminate data races using Swift Concurrency

Join us as we explore one of the core concepts in Swift concurrency: isolation of tasks and actors. We'll take you through Swift’s approach to eliminating data races and its effect on app architecture. We'll also discuss the importance of atomicity in your code, share the nuances of Sendable checking to maintain isolation, and revisit assumptions about ordering work in a concurrent system.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110351", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Task isolation

- Task isolation ensures that data across tasks is not shared in a manner that can introduce data races

### Task

- A task performs a specific job sequentially from start to finish
- Tasks are asynchronous, and their work can be suspended any number of times at `await` operations
- A task is self-contained - has its own resources and can operate by itself, independently of any other task

### Communications between tasks

- done by passing objects across tasks (a task passes an object by returning a value at the end of its body)
- no problem if the shared/transferred data is value type
- can (potentially) cause data races if the data is reference type

#### Sendable

Swift helps us telling us when it's safe to share our data across tasks via the `Sendable` protocol:

- `Sendable` descibes types that can cross an isolation domain (like tasks), without making data races 
- data races checks happen while building by the Swift compiler
- For tasks, the actual `Sendable` constraint comes from their definition: `Task`s return type must conform to `Sendable`
- You should use `Sendable` constraints where you have generic parameters whose values will be passed across different isolation domains
- `Sendable` conformances can be inferred by the Swift compiler for non-public types (but you can add `Sendable` conformance explicitly)
- Classes (reference types) can conform to `Sendable` only under very narrow circumstances
  - e.g., when a `final` class only has immutable storage

- for reference types that do their own internal synchronization (e.g., via locks), you can use `@unchecked Sendable`

```swift
class ConcurrentCache<Key: Hashable & Sendable, Value: Sendable>: @unchecked Sendable {
  var lock: NSLock
  var storage: [Key: Value]

  // ...
}
```

## Actor isolation

- `Actor`s provide a way to isolate state that can be accessed by different tasks, in a coordinated manner that eliminates data races
- an `Actor` is self-contained - has its own resources and can operate by itself, independently of any other Actor
- in order to execute code (or read values) defined in an actor, you need to use a task
- only one task can execute on an actor at a time
- entering into an actor is a potential suspension point, as there might be already another task running on it, and even other tasks waiting to enter into that specific actor 
- the same rules for communications across tasks are true for communication between tasks and actors and between actors
  - said in other words, actors rely on `Sendable`, too

- `Actor`s are reference types, but isolate all of their properties and code to prevent concurrent access
  - all `Actor` types are implicitly `Sendable`

- all Actor instance definitions (properties and functions) are isolated
  - a child/sub task inherits all attributes of the parent task, therefore, if a task is generated directly by an actor function, said task inherits actor isolation from its context (thus will be able to access the actor properties and call other functions without `await`ing on them
  - the same is not true for detached tasks, which do not inherit traits from that task’s originating context
  - Actor properties and functions marked as `nonisolated` are considered to be outside the actor

### @MainActor

- represents main thread
- use it when you need to update UI in your app
- use the `@MainActor` attribute to indicate that the code must run on the main actor:

```swift
@MainActor func updateView() { … }

Task { @MainActor in
  // update UI here
}
```

- the Swift compiler will guarantee that main-actor-isolated code will only be executed on the main thread

- `@MainActor` can also be applied to types, in which case the instances of those types will be isolated to the main actor
  - properties will be only accessible while on the main actor
  - methods are isolated to the main actor, unless marked `nonisolated`

```swift
@MainActor
class ChickenValley: Sendable {
  var flock: [Chicken]
  var food: [Pineapple]

  func advanceTime() {
    for chicken in flock {
      chicken.eat(from: &food)
    }
  }
}
```

## Atomicity

- state can change across `awaits` calls
- if you're not careful, you can end up with a high-level data race where the program is in an unexpected state, even though the data is not corrupted
- when writing your actor, think in terms of synchronous, transactional operations that can be interleaved in any way
- keep async actor operations simple

## Ordering

- Swift Concurrency provides tools for ordering operations
- actors do no guaranteed FIFO processing
- actors execute the highest-priority work first
  - eliminates priority inversions
  - diffent than serial Dispatch queues, which execute in a strictly FIFO order

Tools for ordering:

- `Task`s
- `AsyncStream`s deliver elements in order:

```swift
for await event in eventStream {
  await process(event)
}
```
# Advanced NSOperations

Operations are a flexible way to model your app's business logic, but they can do so much more. See how NSOperation forms the heart of the WWDC app, and how using features like dependencies, readiness, and composition allow you to quickly and easily build dynamic and complex apps.

@Metadata {
   @TitleHeading("WWDC15")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc15/226", purpose: link, label: "Watch Video (44 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Core concepts

- `NSOperationQueue`
  - Wrapper around dispatch queue
  - Makes it easy to cancel operations that have not began executing
  - Lets us specify the max number of concurrent operations, can be changed later (if `max == 1`, this becomes a serial queue)

- `NSOperation`
  - Wrapper around a dispatch block
  - can run several minutes (dispatch blocks too, but it’s not the expected behavior for those)
  - subclassable

## `NSOperation` Lifecycle

![][lifecycleImage]

- Once created the operation starts in a pending state
- After putting it in its queue, the state will move to ready to execute
- Then the operation queue at some point will fetch the operation and begin executing it.
- After the execution finishes, the state will move to finished
- At any point, the operation state can also move to cancelled (unless it has finished already)

### `NSOperation` Cancellation

- defined as a boolean (cancelled) in the `NSOperation` object.
- When we cancel an operation, what the system does is set this boolean
- Up to us (or our subclass) to decide what does it mean for the operation to be cancelled
- Up to us to observe changes in this boolean value and react on it
- Note that there might be scenarios when the cancellation event comes too late and the operation finished before the event reaches the object

### `NSOperation` Ready state

- Again, defined as a boolean (ready).
- Reflects that the operation is ready to execute

### NSOperation dependencies

- Strict ordering
- By default an operation becomes ready only when all its dependencies have been executed
- Not limited by operation queues: an operation can depend on operations in different operation queues
- Be aware of deadlocks! (No mutual dependencies)

### Beyond the basics
- If an operation has dependencies, let that operation create by itself those operations:  
this way we don’t need to create every time multiple levels of dependencies by ourselves, but strictly the dependencies needed by us, and they will take care of everything else. This is super powerful especially for reusability
- It’s ok to add UI things inside the operations
- It’s ok to have dispatch blocks inside the operations
- Add requirements:
  - user location
  - User logged in
  - Internet availability

- These requirements can be reflected by the ready state
- Composing Operations
  - We can also create operations that internally have other operations,
  -  e.g. an object request can be composed by a file download and a file parse
  - e.g. fetch all favorites the internal operation is a loop of fetches that continue until we fetch all the favorites (in case they’re paginated)

- Mutual exclusivity

[lifecycleImage]: WWDC15-226-lifecycle

# Understand and eliminate hangs from your app

Discover how you can track down hangs and delays in your app. We’ll show you tools and methods to discover hangs and their causes, learn about anti-patterns that can lead to hangs, explore best practices for eliminating hangs like GCD, and provide guidance on when you should consider asynchronous code to improve your app performance.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10258", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



What is a hang?

- Whenever the app hangs, is unresponsive, lags, slows down, spins, or is just stuck
- This period of unresponsiveness is called **hang**

What is the app main runloop?

- The main runloop is a loop your application's main thread enters to run event handlers in response to incoming events, primarily user interactions

When a user interacts with an app: 
  
  1. the runloop receives the event
  2. the runloop processes it
  3. then the runloop updates the UI, if needed

- this all happens in one turn of the runloop, and on the main thread
- this process repeats for each user input
- events are buffered and cannot be handled by the main thread during a hang

## What causes a hang?

- The high-level cause for hangs is too much work being done on, or on behalf of, the main thread. 
- To ensure performance, it's important the main thread of your application focuses on what's necessary to update UI

Common cases:

- proactive work in the main thread (e.g., reading, preparing and composite more images than necessary
- performing irrelevant work on the main thread
- use sub-optimal API (e.g. use CPU instead of GPU for rounding corners of views)
- network and I/O on the main thread
- synchronization on the main thread (locks, semaphores and similar)

The main thread services blocks from the main dispatch queue, but it can also service blocks from other queues via dispatch sync:  
anytime a queue dispatch syncs onto another queue, all pending blocks on the other queue have to execute before the newly enqueued one 

## How to diagnose hangs?

During development:

- The time profiler instrument shows you what the app is executing by showing your application’s callstacks over time
- The system trace instrument adds more context with data on system calls, VM faults, I/O, as well as inter and intra-process interactions. 

On production:

- use MetricKit, MetricKit returns a call tree by aggregating callstacks taken during a hang

## How to eliminate hangs?

Reduce work on the main thread:
  
1. optimize work on the main thread, to reduce execution time. 
2. move work off the main thread in a non-blocking manner to keep it responsive

### Caching

- quickly access frequently used assets or previously queried values
- in-memory store, but can be persisted to disk, if needed across multiple app invocations
- formatted assets (e.g. images) are great candidates for caching
- It is important to have an accurate cache invalidation mechanism to strike a balance between having stale data and constantly updating a cache. 
- cache invalidation should happen asynchronously on a secondary dispatch queue to keep the main thread responsive to events
- `NSCache`

### Notification observers

- allow your app to react to changes in a value or state, without having to do expensive, on-demand computation
- to find all observable system notifications, check out the [`NSNotification.Name` documentation][NSNotification.Name]

[NSNotification.Name]: https://developer.apple.com/documentation/foundation/nsnotification/name


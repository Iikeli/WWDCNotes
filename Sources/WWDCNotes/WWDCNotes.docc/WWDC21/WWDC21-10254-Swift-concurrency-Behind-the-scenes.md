# Swift concurrency: Behind the scenes

Dive into the details of Swift concurrency and discover how Swift provides greater safety from data races and thread explosion while simultaneously improving performance. We’ll explore how Swift tasks differ from Grand Central Dispatch, how the new cooperative threading model works, and how to ensure the best performance for your apps.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10254", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(donnywals)
      @GitHubUser(zntfdr)
   }
}



## Threading model

How threads are brought up to handle work on GCD queues: 

- When work is enqueued onto a queue in Grand Central Dispatch, the system will bring up a thread to service that work item
- Since a concurrent queue can handle multiple work items at once, the system will bring up several threads until we have saturated all the CPU cores
- If a thread blocks and there is more work to be done on the concurrent queue, GCD will bring up more threads to drain the remaining work items
  - What does "a thread is blocked" mean? If a thread is blocked, it is suspended by the operating system (OS) (i.e. it is marked as not eligible for running, so that OS scheduler ignores it when selecting threads to run) until the thread can be unblocked - the OS is responsible for scheduling threads
  - By giving your process another thread, we ensure that each cpu core continues to have a thread that executes work at any given time

In GCD, it's easy to have excessive concurrency:

- Overcommitting the system with more threads than CPU cores (e.g., Apple Watch only has 2 cores)
- Risk of thread explosion
- Too many threads come with performance costs:
  - Memory overhead - as each blocked thread is holding onto valuable memory and resources while waiting to run again
  - Scheduling overhead - as new threads are brought up, the CPU need to perform a [full thread context switch][Context_switch] in order to switch away from the old thread to start executing the new thread
    - [timesharing of threads][Time-sharing] - with limited cores and a lot of threads, the scheduling latencies of these threads outweigh the amount of useful work they would do, therefore, resulting in the CPU running less efficiently as well
    - excessive context switching - the cost of swapping threads (for timesharing the cpu) is not negligible

| ![][gcd-timeline] | ![][swift-concurrency-timeline] |
| --- | --- |
| Grand Central Dispatch | Swift Concurrency |

Swift concurrency aims to:

- have one thread running on each cpu core - create only as many threads as there are CPU cores
- replace blocked threads with lightweight objects called <kbd>continuations</kbd> to track resumption of work - this way threads are be able to cheaply and efficiently switch between work items when they are blocked
- no context switches - instead of a full thread context switch, swapping continuations comes has the cost of a function call

In order to achieve this behavior, the operating system needs a runtime contract that threads will not block, and that is only possible if the language is able to provide us with that. Swift's concurrency model and the semantics around it have therefore been designed with this goal in mind.

Two of Swift's language-level features that enable us to maintain this runtime contract:

1. `await` and non-blocking of threads
  - `await` is an asynchronous wait
  - `await` does not block the current thread while waiting for results from the `async` function - Instead, the function may be suspended and the thread will be freed up to execute other tasks

2. the <kbd>Task tree</kbd> - tracking of dependencies in Swift task model

### 1. How `await` is designed to ensure efficient suspension and resumption

How sync functions work:

- Every thread in a running program has one [stack][Call_stack], which it uses to store state for function calls
- When the thread executes a function call, a new frame is pushed onto its stack
- This newly created stack frame can be used by the function to store parameters, local variables, the return address, and any other information that is needed
- Once the function finishes executing and returns, its stack frame is popped

How async functions work:

- like for non-async functions, the stack frame stores local variables that do not need to be available across suspension points (a.k.a. `await` calls)
- async functions will also have an associated frame stored in the [heap][heap]
- async frames (a.k.a. the async function frame in the heap) store information that does need to be available across suspension points
- instead of adding new stack frames across function calls, the top most stack frame is replaced when any variables that will be needed in the future will already have been stored in the list of async frames
- suppose the execution of an `async` function is suspended, and the thread is reused to do some other useful work instead of being blocked
  - since all information that is maintained across a suspension point is stored on the heap, it can be used to continue execution at a later stage
  - this also means that the stack frame can be (and is) safely destroyed
  - this list of async frames is the runtime representation of a <kbd>continuation</kbd>
  - once we resume (in the same or another thread), we create again its stack frame and continue its execution with the info from the heap

- Since Swift continues to use the operating system stack, both `async` and sync Swift code can efficiently call into C and Objective-C
  - C and Objective-C code can also continue efficiently calling sync Swift code

### 2. Swift runtime's tracking of dependencies between tasks

- Functions can be broken up into continuations at an `await`, also known as a potential suspension point:

```swift
let (data, response) = try await URLSession.shared.data(from: feed.url) // Suspension point

let articles = try deserializeArticles(from: data)                      // Continuation
await updateDatabase(with: articles,. for: feed)                        // Continuation (it's ok to have other suspensions points in a continuation)
// ...                                                                  // Continuation
```

- Within a `TaskGroup`, a parent task may create several child tasks and each of those child tasks needs to complete before a parent task can proceed
- These are dependencies tracked by the Swift concurrency runtime
- In Swift, tasks can only await other tasks that are known to the Swift runtime -- be it continuations or child tasks

- Thanks to this clear understanding of the dependency chain between the tasks, the executing thread is able to reason about task dependencies and pick up a different task instead
- Code written with Swift concurrency can maintain a runtime contract that threads are always able to make forward progress
- thanks to this runtime contract, Swift concurrency has built-in OS support, which comes in form of a <kbd>Cooperative thread pool</kbd>
  - This Cooperative thread pool is the default executor for Swift
  - With limited to the number of CPU cores - the thread pool will only spawn as many threads as there are CPU cores
  - Controlled granularity of concurrency
    - Worker threads don't block
    - Avoid thread explosion and excessive context switches

## Things to consider when adopting Swift Concurrency

- performance
  - concurrency comes with costs - such as there will be additional memory allocations and logic execution in the Swift runtime
  - ensure that benefits of concurrency outweighs (OS) costs of managing it
  - profile your code as you adapt Swift concurrency

- `await` and atomicity
  - no guarantee that the thread which executed the code before the `await` will execute the continuation as well (could resume in another thread)
  - Swift concurrency breaks atomicity by voluntarily descheduling the task
  - do not hold locks across `await`s
  - thread-specific data is not hold across `await`s

- preserve the runtime contract (always make forward progress):

| ✅ Safe primitives | ⚠️ Caution required | 🛑 Unsafe primitives |
| --- | --- | --- |
| `await`, `Actor`s, `Task` groups | `os_unfair_lock`, `NSLock` in synchronous code | `DispatchSemaphore`, `pthread_cond`, `NSCondition`, `pthread_rw_loc`, ... |
| Compiler enforced | No compiler support | No compiler support |

> Using a lock in synchronous code is safe when used for data synchronization around a tight, well-known critical section. This is because the thread holding the lock is always able to make forward progress towards releasing the lock. As such, while the primitive may block a thread for a short period of time under contention, it does not violate the runtime contract of forward progress.

> These unsafe primitives hide dependency information from the Swift runtime, but introduce a dependency in execution in your code. Since the runtime is unaware of this dependency, it cannot make the right scheduling decisions and resolve them. In particular, do not use primitives that create unstructured tasks and then retroactively introduce a dependency across task boundaries by using a semaphore or an unsafe primitive.

To help you identify uses of unsafe primitives, run your app with this environment variable (set it in your Xcode run scheme <kbd>Environment variables</kbd> section):

```
LIBDISPATCH_COOPERATIVE_POOL_STRICT=1
```

> This runs your app under a modified debug runtime, which enforces the invariant of forward progress.

## Synchronization

### Mutual exclusion with Actors

Mutual exclusion is guaranteed by actors
- an actor may be executing at most one method call at a time 
- mutual exclusion means that the actor's state is not accessed concurrently, preventing data races

Comparison

|  |  Locks, Serial Queue sync { ...} | Serial Queue async (... } | Actors using cooperative pool |
| No contention (the queue is not already running) | ✅ Reuse thread | ⚠️ Request new thread | ✅ Reuse thread |
| Under contention (the queue is already running) | 🛑 Blocking | ✅ Non-blocking | ✅ Non-blocking |

- When you call a method on an actor that is not running (no contention), the calling thread can be reused to execute the method call
- In the case where the called actor is already running (under contention), the calling thread can suspend the function it is executing and pick up other work

### Actor hopping

- execution switching from one actor to another

Let's say that actor A make an await call to actor B:

- No contention case:
  - the thread can directly hop from actor A to actor B
  - The thread does not block while hopping actors
  - Hopping does not require a different thread
  - The runtime can directly suspend the work item for actor A and create a new work item for the actor B

- Under contention case:
  - the thread cannot hop to actor B, because actor B is already working on another work item (on another thread)
  - a new actor B work item will be created, and will be kept pending
  - actor A will be suspended and the thread it was executing on is now freed up to do other work
  - when the previous actor B work item completes, the runtime may choose to start executing the pending work item for actor A

## Reentrancy and prioritization

Ideally, high-priority work such as that involving user interaction, would take precedence over background work, such as saving backups.  
Actors are designed to allow the system to prioritize work well due to the notion of <kbd>reentrancy</kbd>.

- how actors work under the hood
- how actors compare to existing synchronization primitives you already may be familiar with -- like serial dispatch queues --
- some things to keep in mind when writing code with actors

How GCD Serial dispatch queues reentrancy and prioritization work:

- Dispatch queues execute the items received in a strict first-in, first-out order
- Even if a work item is high-priority, it will need to wait for other work items that came earlier to complete
  - if those earlier work items are low-priority, we're in a [priority inversion situation][Priority_inversion]
  - Serial queues work around priority inversion by boosting/upgrading the priority of all of the work in the queue that's ahead of the high-priority work
  - In practice, this means that the work in the queue will be done sooner; However, this does not resolve the main issue, which is that low-priority items still need to complete before the original high-priority item can start executing
- Actor reentrancy aims to solve this kind of problem

Actor reentrancy:

- actors can execute items in an order that is not strictly first-in, first-out
- actor new work items can make progress, and even complete, while one or more older work items on it are suspended
  - The actor still maintains mutual exclusion, as at most one work item can be executing at a given time
- higher-priority work will be executed first, with lower-priority work following later

## Main actor

- abstracts over an existing notion of the system main thread
- the main thread is disjoint from the threads in the cooperative pool
- `await`ing to and from main actor requires a (thread) context switch
- structure your code so that work for the main actor is batched up, do not jump from/to the main actor continuously (a.k.a. avoid frequently context switching)

[Context_switch]: https://en.wikipedia.org/wiki/Context_switch
[Time-sharing]: https://en.wikipedia.org/wiki/Time-sharing
[Call_stack]: https://en.wikipedia.org/wiki/Call_stack
[heap]: https://en.wikipedia.org/w/index.php?title=Heap_(memory_management)
[Priority_inversion]: https://en.wikipedia.org/wiki/Priority_inversion
[gcd-timeline]: WWDC21-10254-gcd-timeline
[swift-concurrency-timeline]: WWDC21-10254-swift-concurrency-timeline
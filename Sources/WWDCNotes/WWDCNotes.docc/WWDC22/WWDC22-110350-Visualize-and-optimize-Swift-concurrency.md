# Visualize and optimize Swift concurrency

Learn how you can optimize your app with the Swift Concurrency template in Instruments. We'll discuss common performance issues and show you how to use Instruments to find and resolve these problems. Learn how you can keep your UI responsive, maximize parallel performance, and analyze Swift concurrency activity within your app.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110350", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Swift Concurrency Recap

### `Async/await` 

- basic syntactic building blocks for concurrent code
- allow you to create and call functions that can suspend their work in the middle of execution, then resume that work later, without blocking an execution thread

### `Task`s

- basic unit of work in concurrent code
- execute concurrent code and manage its state and associated data
- contain local variables
- handle cancellation
- begin/suspend execution of async code

### Structured concurrency

- makes it easy to spawn child tasks to run in parallel and wait for them to complete
- ensures that tasks are awaited or automatically canceled if not used

### `Actor`s

- coordinate multiple tasks that need to access shared data
- isolate data from the outside
- allow only one task at a time to manipulate their internal state, avoiding data races from concurrent mutation

### `Continuation`s 

- bridge between Swift concurrency and other forms of async code (GCD)
- a continuation suspends the current task and provides a callback which resumes the task when called
- can be used with callback-based async APIs
- Continuation callbacks must be called exactly once

## Concurrency Optimization

Even with Swift concurrency is still possible to write code that misuses concurrency constructs, common issues:

### Main actor blocking 

- occurs when a long-running task runs on the main Actor
- can cause your app to hang

### Actor contention 

- occurs when several tasks attempt to use the same `Actor` simultaneously
- each task must wait for the Actor to become available
- the `Actor` serializes execution of those tasks
- hurt performance by reducing parallel execution
- To fix this, we need make sure that tasks only run on the Actor when they really need exclusive access to the Actor's data. Everything else should run off of the Actor

### Thread pool exhaustion

- hurt performance by reducing parallel execution
- can hurt performance or even deadlock an application
- happens when a task waits for something via a blocking call, such as blocking file or network IO, or acquiring locks, without suspending
- this breaks the requirement for tasks to make forward progress, because the task continues to occupy the thread where it's executing, but it isn't actually using a CPU core
- in such situations the concurrency runtime is unable to fully use all CPU cores, and can even cause deadlock
- make blocking calls outside Swift Concurrency (for example by running it on a Dispatch queue, and bridge it to the concurrency world using continuations)

### Continuation misuse

- occurs when the continuation callback is called multiple times or never
- the program will crash or misbehave when the continuation callback is called more than once
- the task will leak (a.k.a. wait forever) if the callback is never called

The new Swift Concurrency instrument helps catching these problems in your app.
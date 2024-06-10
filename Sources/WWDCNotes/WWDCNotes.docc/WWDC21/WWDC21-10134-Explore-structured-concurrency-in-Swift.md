# Explore structured concurrency in Swift

When you have code that needs to run at the same time as other code, it’s important to choose the right tool for the job. We'll take you through the different kinds of concurrent tasks you can create in Swift, show you how to create groups of tasks, and find out how to cancel tasks in progress. We'll also provide guidance on when you may want to use unstructured tasks.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10134", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Tasks

- You can create additional tasks to add concurrency to a program
- A task provides a fresh execution context to run asynchronous code
- Each task runs concurrently with respect to other execution contexts
- Calling an `async` function does not create a new task

## Async-let tasks

- When the process encounter an `async let` statement, a child task is created, while the main/parent task continues running
- The parent task will suspend (if needed) only when it needs to get the result from the `async let` child task, and it does so by using the (`try`) `await` keyword. In other words, the parent task might suspend when it start using the variables that are concurrently bound

```swift
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
  let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
  async let (data, _) = URLSession.shared.data(for: imageReq) // 👈🏻 async
  async let (metadata, _) = URLSession.shared.data(for: metadataReq) // 👈🏻 async
  guard let size = parseSize(from: try await metadata), // 👈🏻 await
        let image = try await UIImage(data: data)?.byPreparingThumbnail(ofSize: size) // 👈🏻 await
  else {
    throw ThumbnailFailedError()
  }
  return image
}
```

### Task Tree

- keeps track of tasks and their children
- influences the attributes of your tasks like cancellation, priority, and task-local variables
- a child/sub task inherits all attributes of the parent task
- whenever you make a call from one async function to another, the same task is used to execute the call
- Tasks are not the child of a specific function, but their lifetime may be scoped to it
- A task parent-child link enforces a rule that says a parent task can only finish its work if all of its child tasks have finished
  - let's say that we have two child tasks and one of them errors out, causing the parent, which was `try await`ing on it, to throw an error: the tree is responsible to cancel other child tasks and then await for them to finish before the parent task function can exit/throw
  - Marking a task as canceled does not stop the task. It simply informs the task that its results are no longer needed
  - When a task is canceled, all subtasks that are decedents of that task will be automatically canceled, too

#### Task cancellation is cooperative

- Tasks are not stopped immediately when cancelled
- Cancellation can be checked from anywhere (async or not)
- Design your code with cancellation in mind

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
  var thumbnails: [String: UIImage] = [:]
  for id in ids {
    try Task.checkCancellation() // 👈🏻 cancellation check, this call throws an error if the current task has been canceled
    thumbnails[id] = try await fetchOneThumbnail(withID: id)
  }
  return thumbnails
}
```

You can also check for cancellation without throwing:

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
  var thumbnails: [String: UIImage] = [:]
  for id in ids {
    if Task.isCancelled { break } // 👈🏻 cancellation check
    thumbnails[id] = try await fetchOneThumbnail(withID: id)
  }
  return thumbnails // 👈🏻 In case of cancellation, we return a partial result
}
```

## Group tasks

- A task group is a form of structured concurrency that is designed to provide a dynamic amount of concurrency
- You can introduce a task group by calling the `withThrowingTaskGroup` function
- This function gives you a scoped group object to create child tasks that are allowed to throw errors
- Tasks added to a group cannot outlive the scope of the block in which the group is defined
- You create child tasks in a group by invoking its `async(_:)` method
- Child tasks added this way will begin executing immediately and in any order
- When the group object goes out of scope, the completion of all tasks within it will be implicitly awaited

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
  var thumbnails: [String: UIImage] = [:]
  try await withThrowingTaskGroup(of: Void.self) { group in // 👈🏻
    for id in ids {
      group.async {
        // Error: Mutation of captured var 'thumbnails' in concurrently executing code
        thumbnails[id] = try await fetchOneThumbnail(withID: id)
      }
    }
  }
  return thumbnails
}
```

In the sample code above we have a data race issue:  
the `thumbnails` dictionary cannot handle more than one access at a time, and if two child tasks tried to insert thumbnails simultaneously, that could cause a crash or data corruption.

- Whenever you create a new task, the work that the task performs is within a new closure type called a `@Sendable` closure. 
- The body of a `@Sendable` closure is restricted from capturing mutable variables in its lexical context, because those variables could be modified after the task is launched.
- This means that the values you capture in a task must be safe to share. E.g., value types, or objects designed to be accessed from multiple threads, like actors, and classes that implement their own synchronization.

A way to solve this in our example, is to have each child task return a value:  
this design gives the parent task the sole responsibility of processing the results. 

```swift
func fetchThumbnails(for ids: [String]) async throws -> [String: UIImage] {
  var thumbnails: [String: UIImage] = [:]
  try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
    for id in ids {
      group.async {
        return (id, try await fetchOneThumbnail(withID: id)) // 👈🏻 return only
      }
    }
    // Obtain results from the child tasks, sequentially, in order of completion.
    for try await (id, thumbnail) in group {
      thumbnails[id] = thumbnail // 👈🏻 assign to the dictionary from the parent task
    }
  }
  return thumbnails
}
```

Task tree differences from async let tasks:

- when your group goes out of scope through a normal exit from the block. Then, cancellation for child tasks is not implicit
  - This behavior makes it easier for you to express the fork-join pattern using a task group, because the jobs will only be awaited, not canceled
- You can also manually cancel all tasks before exiting the block using the group’s `cancelAll` method

## Unstructured Tasks

- give you a lot more flexibility at the expense of needing a lot more manual management
- Useful when:
  - some tasks need to launch from non-async contexts
  - some tasks live beyond the confines of a single scope

Characteristics:

- Inherit actor isolation and priority of the origin context
- Lifetime is not confined to any scope
- Can be launched anywhere, even non-async functions
- Must be manually cancelled or awaited

Example:

```swift
@MainActor
class MyDelegate: UICollectionViewDelegate {
  var thumbnailTasks: [IndexPath: Task<Void, Never>] = [:]
  
  func collectionView(_ view: UICollectionView, willDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
    let ids = getThumbnailIDs(for: item)
    thumbnailTasks[item] = Task { // 👈🏻 create and store unstructured tasks
      defer { thumbnailTasks[item] = nil } // 👈🏻 we remove the task when it's finished, so we don't cancel it when it's finished already
      let thumbnails = await fetchThumbnails(for: ids)
      display(thumbnails, in: cell)
    }
  }
  
  func collectionView(_ view: UICollectionView, didEndDisplay cell: UICollectionViewCell, forItemAt item: IndexPath) {
    thumbnailTasks[item]?.cancel() // 👈🏻 we cancel said task when that cell is no longer displayed
  }
}
```

Note here that we can access the same dictionary inside and outside of that async task without getting a data race flagged by the compiler. Our delegate class is bound to the main actor, and the new task inherits that, so they’ll never run together in parallel. 

## Detached Tasks

- Unstructured tasks inherit traits from that task’s originating context, detached tasks don't.
- maximum flexibility
- Unscoped lifetime, manually cancelled and awaited
- Do not inherit anything from their originating context - e.g., they're not constrained to the same actor and don’t have to run at the same priority as where they were launched
- Detached tasks run independently with generic defaults for things like priority, but they can also be launched with optional parameters to control how and where the new task gets executed

## Task Cheat sheet

|  | Launched by | Launchable from | Lifetime | Cancellation | Inherits from origin |
| --- | --- | --- | --- | --- | --- |
| async-let tasks | `asvnc let x` | `async` functions | scoped to statement | automatic | priority, task-local values |
| Group tasks | `group.async` | `withTaskGroup` | scoped to task group | automatic | priority, task-local values |
| Unstructured tasks | `Task` | anywhere | unscoped | via `Task` | priority, task-local values, actor |
| Detached tasks | `Task.detached` | anywhere | unscoped | via `Task` | nothing |

# Meet async/await in Swift

Swift now supports asynchronous functions — a pattern commonly known as async/await. Discover how the new syntax can make your code easier to read and understand. Learn what happens when a function suspends, and find out how to adapt existing completion handlers to asynchronous functions.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10132", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- When you call an asynchronous function, it unblocks your thread quickly, having kicked off its work. That allows the thread to do other things while that long running work completes.
- in functions, properties, and initializers, `await` can be used on expressions to indicate where the function might unblock the thread

## Async Sequences

An async sequence is just like a normal sequence except that it vends its elements asynchronously. So fetching the next item must be marked with the await keyword, indicating that it’s async.

```swift
for await id in staticImageIDsURL.lines {
  let thumbnail = await fetchThumbnail(for: id)
  collage.add(thumbnail)
}
let result = await collage.draw()
```

As the function iterates over the async sequence, over and over, it may unblock the thread while awaiting the next element and then resume either with the next element into the body of the loop or, if there are no elements left, after the loop.

## Suspension

The `await` keyword indicates that your async function might suspend there. 

What does it mean for an async function to suspend?

- For sync functions, when called, you hand your function’s thread control over to that function. Then the thread will be fully occupied doing work on behalf of that one function until it finishes
- For asynchronous functions, when called, it _can_ give up control of the thread by suspending. When an async function suspends, it gives up control of the thread. But rather than giving control back to your function, it gives control of the thread to the system. The system is then free to use the thread to do other work.
- a function can suspend itself as many times as it needs to

## Takeaways

- `async` enables a function to suspend - when a function suspends itself, it suspends its callers too. So its callers must be async as well
- `await` marks where a function _may_ suspend
- other work can happen during a suspension - the thread is not blocked
- once an awaited async call completes, execution resumes after the await

## Bridging from sync to async

When you want to call an async function from a sync function, you can use an async task function:

```swift
struct ThumbnailView: View {
  ...

  var body: some View {
    Image(uiImage: self.image ?? placeholder)
      .onAppear { // 👈🏻 on appear accepts a sync function
        Task { // 👈🏻 async task
          self.image = try? await self.viewModel.fetchThumbnail(for: post.id)
        }
      }
  }
}
```

An async task packages up the work in the closure and sends it to the system for immediate execution on the next available thread, like the async function on a global dispatch queue.

## Async alternatives and continuations

When you have a sync function and would like to create an async alternative, use `withCheckedThrowingContinuation`:

```swift
// Existing function
func getPersistentPosts(completion: @escaping ([Post], Error?) -> Void) {     
  do {
    let req = Post.fetchRequest()
    req.sortDescriptors = [NSSortDescriptor(key: "date", ascending: true)]
    let asyncRequest = NSAsynchronousFetchRequest<Post>(fetchRequest: req) { result in
      completion(result.finalResult ?? [], nil)
    }
    try self.managedObjectContext.execute(asyncRequest)
  } catch {
    completion([], error)
  }
}

// Async alternative
func persistentPosts() async throws -> [Post] {     
  return try await withCheckedThrowingContinuation { (continuation: CheckedContinuation<[Post], Error>) in
    self.getPersistentPosts { posts, error in
      if let error = error { 
        continuation.resume(throwing: error) 
      } else {
        continuation.resume(returning: posts)
      }
    }
  }
}
```
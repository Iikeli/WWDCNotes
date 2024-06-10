# Introducing Combine

Combine is a unified declarative framework for processing values over time. Learn how it can simplify asynchronous code like networking, key value observing, notifications and callbacks.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/722", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What’s Combine?

- A unified, declarative API for processing values over time.
- Combine was designed for composition.

## Key Concepts

### `Publisher`

- it's a protocol
- Declarative part of Combine.
- Describes how values/errors are produced.
- Sends (sequences of) values over time.
- Two generic types: `Output` and `Error`
- Completion via `.finished` or `.failure`
- Publishers are not necessary who produced the value/error.

### `Subscriber`

- it's a protocol
- `Publisher` counterpart
- Receives values from publishers
- Two generic types: `Input` and `Error`
- Can cancel a subscription
- `Subscription`: A subscription is how a `Subscriber` controls the flow of data from a Publisher to the `Subscriber`.

### Operators

- In-between `Publisher`s and `Subscriber`s, they transform/filter the published value into something desired by the subscriber
- Operators forward errors, only change values.
- They also manage scheduling/time, thread/queue movement, and more

Combine provides a lot of operators:  
`catch`, `dropFirst`, `allSatisfy`, `breakpoint`, `setFailureType`, `prepend`, `replaceError`, `append`, `map`, `filter`, `count`, `abortOnError`, `breakpointOnError`, `ignoreOutput`, `first`, `min`, `last`, `output`, `prefix`, `replaceEmpty`, `compactMap`, `zip`, `flatMap`, `removeDuplicates`, `reduce`, `combineLatest`, `collect`, `retry`, `contains`, `switchToLatest`, `replaceNil`, `scan`, and more!

The idea is that operators are composable:  
instead of having a few operators that do a lot, Combine provides many operators that do a little, then it’s up to us to choose which operators to use based on our needs.

If a operator returns `nil`, it’s like stopping the event/value to proceed further. We can use most of the functional operators we’re used with arrays: prefix, compactMap, filter...

`zip` converts multiple inputs (from different publishers) into a single tuple. It requires all the inputs (from all upstreams) to be delivered before sending the data downstream. Therefore it acts as a when/and operation.

`combineLatest` is like `zip`, but it keeps the latest value of each upstream. Whenever a new value comes in, a new event is sent downstream with all old values except the new one.

Note that both zip and CombineLatest can transform the values into something else (no need to forward necessarily the tuple)

### Future vs Publisher

- If we need to represent a single value asynchronously, we have a future. 
- If we need to represent many values asynchronously, that's a Publisher. 
- Future is a real type in combine

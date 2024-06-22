# Combine in Practice

Expand your knowledge of Combine, Apple's new unified, declarative framework for processing values over time. Learn about how to correctly handle errors, schedule work and integrate Combine into your app today.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/721", purpose: link, label: "Watch Video (34 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



In this session more Combine operators are introduced:

- `map`, `tryMap`, `decode`, `assertNoFailure`, `retry`, `catch`, `mapError`, `setFailureType` ...

- `Just`: special `Publisher` for when you already have a value that you want to publish. It’s called `Just`, as in just publish this value.

## Scheduled Operators

- Describes when and where a particular event is delivered
- Supported by `RunLoop` and `DispatchQueue`
- Delay: defers the delivery of an event until some future time. 
- Throttle: guarantees that events are delivered no faster than a specified rate.
- `Receive(on:)`: guarantees that downstream received events will be delivered on a particular thread or queue. 

## Subscriber

Subscribers have three functions:

1. `receive(subscription:)` when subscribing the publisher will call this function exactly once
2. `receive(_:)` the publisher will provide 0 or more values via this method
3. `receive(completion:)` the publisher can eventually provide a completion only once, meaning that it has completed or a failure has risen

Once `receive(completion:)` is called, no further value will be forwarded.

## Special Subscriptions

### `Sink`

Just provide a closure and now for every value received, the closure is going to get called with the value.

This `Sink` will return a `canceller`, which is like a token where we can call cancel when we no longer want to get new values.

### `Subjects`

Behaves like Publisher and Subscriber.

Supporting multi casting a single value (broadcast to multiple subscribers)

Two kinds of subjects:

- `Passthrough`: doesn’t store any value, therefore we’ll get a value only once a new one will be sentWe can inject a passthrough subjects in a stream by calling `onePublisher.share()`
- `CurrentValue`: like `Passthrough`, however it stores the last value, so new subscribers have an opportunity to catch up

## SwiftUI

- SwiftUI owns the subscriber
- We only need to provide a publisher

### `@BindableObject`

- `@BindableObject`s in SwiftUI have a single associated type. 
- It's a Publisher that is constrained to never fail.

Example:

```swift
class WizardModel: BindableObject { 
	var trick: WizardTrick { didSet { didChange.send() }
	var wand: Wand? { didSet { didChange.send() } 

	let didChange PassthroughSubject<Void, Never>()
}

struct TrickView: View { 
  @ObjectBinding var model: WizardModel 

  var body: some View { 
  	Text(model.trick.name)
  }
}
```

- `@ObjectBinding` allows SwiftUI to automatically discover and subscribe to our Publisher.
- SwiftUI will automatically generate a new body whenever we signal that the model has changed. 
- Note how the “combine upstream” value is `Void`, we do not use as the `View` owns the model anyway, so we can access it from there instead.

### `@Published`

- Adds a publisher to a property
- Access the the published via wrapped value (in the example below, `$password` is the publisher aka wrapped value)

```swift
@Published var password: String 

self.password = "1234"

let currentPassword: String = self.password

let printerSubscription = $password.sink {
	print("The published value is '\($0)'")
} 

self. password = "password" 
```

### `eraseToAnyPublisher`

Advertise the exact contract we want for our API boundary and hide all the implementation details along the way.

Example:

```swift
@Published var username: String = "" 

var validatedUsername: AnyPublisher<String, Never> {
	return $username 
    .debounce(for: 0.5, scheduler: RunLoop.main)
    .removeDuplicates()
    .eraseToAnyPublisher() 
}
```

Further example with Future:

```swift
@Published var username: String = ""

var validatedUsername: AnyPublisher<String?, Never> { 
	return $username 
    .debounce (for: 0.5, scheduler: RunLoop.main) 
    .removeDuplicates()
    .flatMap { username in 
    	return Future { promise in 
        self.usernameAvailable(username) { available in
        	promise (.success(available ? username : nil))
        }
      }
    } 
    .eraseToAnyPublisher() 
}
```

### `AnyCancellable`

- What is returned after creating a subscriber
- This class automatically calls `cancel()` on `deinit`. 
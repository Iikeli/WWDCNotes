# Design protocol interfaces in Swift

Learn how you can use Swift 5.7 to design advanced abstractions using protocols. We'll show you how to use existential types, explore how you can separate implementation from interface with opaque result types, and share the same-type requirements that can help you identify and guarantee relationships between concrete types.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110353", purpose: link, label: "Watch Video (25 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



We will use this protocol definition as an example:

```swift
protocol Animal {
  associatedtype CommodityType: Food
  associatedtype FeedType: AnimalFeed

  func produce() -> CommodityType
  func eat(_: FeedType)
  var isHungry: Bool { get }
}
```

## Understand type erasure

- when you call a method returning an associated type on an existential type, the compiler will use type erasure to determine the result type of the call
- type erasure replaces these associated types with corresponding existential types that have equivalent constraints

### Type erasure semantics

#### Producing position

- `associatedtype`s appearing in the result of a protocol method declaration are in <kbd>producing position</kbd>, because calling the method will produce a value of this type
  - e.g., the `produce()` return type in the `Animal` protocol definition above

- The type `any Food` is called the <kbd>upper bound</kbd> of the associated `CommodityType`

- the actual concrete type that is returned from `produce()` can safely convert to the upper bound:

```swift
let animals: [any Animal] = [Cow()]

animals.map { animal in
  animal.produce() // we're calling `produce()` on an `any Animal` that holds a `Cow` at runtime.
}
```

#### Consuming position

- `associatedtype`s appearing in the parameter list of a protocol method declaration are in <kbd>consuming position</kbd>, because calling the method will produce a value of this type

- the upper bound cannot safely convert to the actual concrete type, because the concrete type is unknown

```swift
let animals = [Cow()]

animals.map { animal in
  animal.eat(???) // given an arbitrary `any AnimalFeed`, there is no way to statically guarantee that it stores what Cow needs
}
```

- type erasure does not allow us to work with associated types in consuming position
- instead, you must unbox the existential `any` type by passing it to a function that takes an opaque `some` type

## Hide implementation details

### Constrained opaque result type

- new in swift 5.7
- written by applying type arguments in angle brackets after the protocol name - e.g., `some Collection<any Animal>`

## Identify type relationships

- every protocol has a `Self` type, which stands for the concrete conforming type
- we can express the relationship between  `associatedtype`s using a same-type requirement, written in a `where` clause
- A same-type requirement expresses a static guarantee that two different, possibly nested associated types must in fact be the same concrete type

```swift
protocol AnimalFeed{
  associatedtype CropType: Crop
    where CropType.FeedType == Self // 👈🏻 same-type requirement
  
  static func grow() -> CropType
}

protocol Crop {
  associatedtype FeedType: AnimalFeed
    where FeedType.CropType == Self // 👈🏻 same-type requirement
  func harvest() -> FeedType
```

- By understanding your data model, you can use same-type requirements to define equivalences between these different nested associated types
- Generic code can then rely on these relationships when chaining together multiple calls to protocol requirements
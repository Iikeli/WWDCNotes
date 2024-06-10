# ARC in Swift: Basics and beyond

Learn about the basics of object lifetimes and ARC in Swift. Dive deep into what language features make object lifetimes observable, consequences of relying on observed object lifetimes and some safe techniques to fix them.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10216", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- prefer value types to avoid the dangers of unintended sharing that comes with reference types

## Object lifetimes and ARC

- An object’s lifetime in Swift begins at initialization (`init()`) and ends at last use
- ARC automatically manages memory, by deallocating an object after its lifetime ends
- ARC determines an object’s lifetime by keeping track of its reference counts
- ARC is mainly driven by the Swift compiler which inserts retain and release operations
- At runtime, retain increments the reference count and release decrements it
- When the reference count drops to zero, the object will be deallocated

## Example

Let's say that we have the following code:

```swift
class Traveler {
  var name: String
  var destination: String?
}

func test() {
  let traveler1 = Traveler(name: "Lily")
  let traveler2 = traveler1
  traveler2.destination = "Big Sur"
  print("Done traveling")
}
```

In `test()`, first, a `Traveler` object is created, then its reference is copied, and finally, its destination is updated

In order to automatically manage the memory of the `Traveler` object, the Swift compiler inserts:

- a <kbd>retain</kbd> operation when a reference begins
- a <kbd>release</kbd> operation after the last use of the reference

Note how in `test()`:

1. `traveler1` is the first reference to the `Traveler` object, and its last use is the copy
2. `traveler2` is another reference to the `Traveler` object, and its last use is the destination update

In `test()`'s body the Swift compiler: 

- inserts a <kbd>release</kbd> operation immediately after the last use of the `traveler1` reference
- it does **not** insert a <kbd>retain</kbd> operation when the reference begins, because initialization sets the reference count to <kbd>1</kbd>
- inserts a <kbd>retain</kbd> operation when the `traveler2` reference begins
- inserts a <kbd>release</kbd> operation immediately after the last use of the `traveler2` reference

![][compilerReference]

Let's step through the code and see what happens at runtime:

- the `Traveler` object is created on the heap and initialized with a reference count of <kbd>1</kbd>
- in preparation of the new `traveler2` reference, the <kbd>retain</kbd> operation (added by the compiler) executes, incrementing the reference count to <kbd>2</kbd>
- after the last use of the `traveler1` reference, the <kbd>release</kbd> operation executes, decrementing the reference count to <kbd>1</kbd>
- after the last use of the `traveler2` reference, the <kbd>release</kbd> operation executes, decrementing the reference count to <kbd>0</kbd>
- Once the reference count drops to zero, the object can be deallocated

![][runtimeReference]

## Observable object lifetimes

- Object lifetimes in Swift are use-based
- An object's guaranteed _minimum_ lifetime begins at initialization and ends at last use

> This is different from languages like C++, in which an object’s lifetime is guaranteed to end at the closing brace

- in the example above, we saw the object was deallocated immediately after the last use, however, in practice, object lifetimes are determined by the retain and release operations inserted by the Swift compiler
- depending on the ARC optimizations, the observed object lifetimes may differ from their guaranteed minimum, ending beyond the last use of the object
  - In such cases, the object is deallocated at a program point beyond its last use

## Deinitialization side effects

In most cases, it doesn’t matter what the exact lifetime of an object is. However, with language features like `weak` and `unowned` references and deinitializer side effects, it is possible to observe object lifetimes:

- if you have programs that rely on observed object lifetimes instead of guaranteed object lifetimes, you can end up with problems in the future
- because relying on observed object lifetimes may work today, but it is only a coincidence
- Observed object lifetimes are an emergent property of the Swift compiler and **can change as implementation details change**

## Safe techniques for using `weak` and `unowned` references

- Swift's `withExtendedLifetime()` - explicitly extends the lifetime of an object
  - With this approach, you should ensure `withExtendedLifetime()` is used every time a weak reference has a potential to cause bugs

- Redesign to access via `strong` reference
- Redesign to avoid `weak`/`unowned` reference

Reference cycles can often be avoided by rethinking algorithms and transforming cyclic class relationships to tree structures.

Avoiding the need for weak and unowned references may have additional implementation cost, but this is a definite way to eliminate all potential object lifetime bugs.

## Safe techniques for using deinitializer side-effects

- Swift's `withExtendedLifetime()`
- Redesign to limit visibility of internal class details
- Redesign to avoid deinitializer side-effects (e.g., use `defer` instead of `deinit`)

## Optimize Object Lifetimes

- New in Xcode 13, a new experimental <kbd>Optimize Object Lifetimes</kbd> build setting is available for the Swift compiler
- This enables powerful lifetime shortening ARC optimizations
- With this build setting turned on, you may see objects being deallocated immediately after last use much more consistently, bringing observed object lifetimes closer to their guaranteed minimum
- This may expose hidden object lifetime bugs

[compilerReference]: WWDC21-10216-compile-time
[runtimeReference]: WWDC21-10216-runtime
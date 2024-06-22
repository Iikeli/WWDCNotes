# Protocol-Oriented Programming in Swift

At the heart of Swift's design are two incredibly powerful ideas: protocol-oriented programming and first class value semantics. Each of these concepts benefit predictability, performance, and productivity, but together they can change the way we think about programming. Find out how you can apply these ideas to improve the code you write.

@Metadata {
   @TitleHeading("WWDC15")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc15/408", purpose: link, label: "Watch Video (45 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



## Classes are Awesome?

Classes give us:

- Encapsulation
- Access control
- Abstraction
- Namespaces
- Expressive syntax
- Extensibility

Access control, abstraction, and namespacing help us manage complexity.

**We can do all of the above with structs and enums too.** So, *types* are awesome.

### Inheritance Hierarchies

![][inheritance_hierarchies]

### Customization points and reuse

![][customization_reuse]

A superclass can define a function, and subclasses get that functionality for free. The magic happens when the author breaks out a small part of that operation into a separate customization point.

The subclass can then override this customization point. Subclasses can reuse difficult logic while maintaining open-ended flexibility.

## 3 Problems with Classes

### 1. Implicit sharing

Let's look at a common scenario. `A` and `B` share some data, but `A` cannot modify this data without affecting `B`:

![][problem_implicit_sharing]

Programmers can defensively make copies of data, but that leads to inefficiency. 

Also, modifying data from different threads can lead to race conditions, and defensively adding locks leads to further inefficiency + deadlocks.

All of this leads to further complexity--in a word, **bugs**.

One effect of implicit sharing on Cocoa platforms:

> It is not safe to modify a mutable collection while enumerating through it. Some enumerators may currently allow enumeration of a collection that is modified, but this behavior is not guaranteed to be supported in the future.

**This does not apply to Swift because all Swift collections are value types**. The collection we are iterating over and the one we are modifying are distinct.

### 2. Class inheritance is too intrusive

Class inheritance is monolithic--we get only 1 superclass. What if we want to model multiple abstractions? For example, what if our class wants to be a collection and be serialized? We can't if both `Collection` and `Serialized` are classes.

Classes also get bloated as everything that *might* be related gets thrown together.

We also have to choose our superclass at the moment that we define our class, not later in some extension.

What if our superclass had stored properties? We have to accept them, *and* we have to initialize them even if we don't need them. We also need to do this without breaking any of the superclass's [invariants](https://softwareengineering.stackexchange.com/a/355642/355782).

Finally, it's natural for authors to write their code as if they know what to override (and more importantly, what *not* to override). For example, they might not use `final` and they might not account for a method being overwritten.

**This is why Cocoa programmers increasingly use the delegate pattern.**

### 3. Lost type relationships

If we want to write a generalized sort or binary search, we need a way to compare 2 elements. With classes, we end up with something like this:

```swift
class Ordered {
  func precedes(other: Ordered) -> Bool
}

func binarySearch(sortedKeys: [Ordered], forKey k: Ordered) -> Int {
  var low = 0, high = sortedKeys.count

  while high > low {
    let mid = low + (high - low) / 2

    if sortedKeys[mid].precedes(k) {
      low = mid + 1
    } else {
      high = mid
    }
  }

  return low
}
```

Of course, we can't just define a class function without filling in it's body. What can we put there if we don't know anything about an arbitrary instance of `Ordered`? There's really nothing we can do other than error out:

```swift
class Ordered {
  func precedes(other: Ordered) -> Bool {
    fatalError("implement me!")
  }
}
```

This is the first sign that we're fighting the type system. We can't just brush this aside saying that "as long as each subclass of `Ordered` implements `precedes`, we'll be ok".

Let's say we move along and implement a subclass of `Ordered`:

```swift
class Number: Ordered {
  var value: Double = 0

  override func precedes(other: Ordered) -> Bool {
    return value < other.value  // ERROR: 'Ordered' does not have a member named 'value'.
  }
}
```

This won't compile because `Ordered` could be anything, such as a `Label` class. Now we need to downcast to get to the right type:

```swift
return value < (other as! Number).value
```

What if `other` turns out to be a `Label`? Our code would trap. We run into this error because classes don't let us express the crucial type relationship between the type of `self` and the type of `other`.

In fact, any time we see `as! Subclass` in our code, we should treat it as a code smell. This is a sign that a type relationship was lost, usually due to using classes for abstraction.

## We need a better abstraction mechanism

We need a mechanism that that:

- Supports value types and classes
- Supports static type relationships and dynamic dispatch
- Is non-monolithic
- Supports retroactive modeling (can conform to another type's requirements in an extension)
- Doesn't impose instance data on models
- Doesn't impose initialization burdens on models
- Makes clear what to implement/override

## The answer: Protocols

Swift is the first protocol-oriented programming language. From the way `for` loops and string literals work to the emphasis in the standard library on generics, at its heart, Swift is protocol-oriented.

### Start with a protocol, not a class

Let's refactor our last example. Protocols don't allow method bodies, and there's nothing to override. Lastly, a `Number` doesn't need to be a reference type, so we can change it to a `struct`:

```swift
protocol Ordered {
  func precedes(other: Ordered) -> Bool
}

struct Number: Ordered {
  var value: Double = 0

  func precedes(other: Ordered) -> Bool {
    return self.value < (other as! Number).value
  }
}
```

We still have the static type safety hole when we downcast to `Number`. We can't drop the typecase by changing `Ordered` to `Number` because then `Number` wouldn't conform to `Ordered`:

```swift
func precedes(other: Number) -> Bool {
  // ERROR: protocol requires function 'precedes' with type '(Ordered) -> Bool'
  // candidate has non-matching type '(Number) -> Bool'
  return self.value < other.value
}
```

To make this code compile, we need to add a `Self` requirement to the protocol. `Self` is a placeholder for the dynamic type that conforms to the protocol:

```swift
protocol Ordered {
  func precedes(other: Self) -> Bool
}
```

Now we can refactor our binary search. When `Ordered` was a class, our function declaration looked like:

```swift
func binarySearch(sortedKeys: [Ordered], forKey k: Ordered) -> Int { ... }
```

Notice that, when `Ordered` was a class, our array of `Ordered` could contain any subclass. For example, we could have an array of `[Number, Label, Fruit]` if all 3 were subclasses of `Ordered`.

After changing `Ordered` to a protocol and adding the `Self` requirement, our array is expected to only contain one conforming type. We get the following error:

> protocol 'Ordered' can only be used as a generic constraint because it has Self or associated type requirements.

To fix the compiler error, we have to declare that our function will only work on any homogeneous array of type `T` which conforms to `Ordered`:

```swift
func binarySearch<T: Ordered>(sortedKeys: [T], forKey k: T) -> Int { ... }
```

Did we lose flexibility by forcing our array to be homogeneous? Not at all! Earlier, we handled the homogeneous case by trapping:

```swift
class Ordered {
  func precedes(other: Ordered) -> Bool {
    fatalError("implement me!")
  }
}
```

A homogeneous array is what we wanted in the first place.

### Two worlds of protocols

| Without Self Requirement  |  With Self Requirement |
|---|---|
| `func precedes(other: Ordered) -> Bool` | `func precedes(other: Self) -> Bool` |
| Usable as a type <br/> `func sort(inout a: [Ordered])` | Only usable as a generic constraint <br/> `func sort<T: Ordered>(inout a: [T])` |
| Think "heterogeneous" | Think "homogeneous" |
| Every model must deal with all others | Models are free from interaction |
| Dynamic dispatch | Static dispatch |
| Less optimizable | More optimizable |

## Replacing classes in OOP with protocols

We're going to model a diagramming app where users can drag and drop shapes on a drawing surface and then interact with those shapes. We're building the document and display model.

First, a primitive "renderer" which just prints out drawing commands:

```swift
struct Renderer {
  func move(to point: CGPoint) {
    print("Move to (\(point.x), \(point.y))")
  }

  func line(to point: CGPoint) {
    print("Line to (\(point.x), \(point.y))")
  }

  func arc(at center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat) {
    print("Arc at \(center), radius: \(radius), startAngle: \(startAngle), endAngle: \(endAngle)")
  }
}
```

Next, a `Drawable` protocol which provides a common interface for all our drawing elements:

```swift
protocol Drawable {
  func draw(using renderer: Renderer)
}
```

Then, shapes like polygons. Notice that this is a value type built out of another value type, `CGPoint`:

```swift
struct Polygon: Drawable {
  var corners = [CGPoint]()

  func draw(using renderer: Renderer) {
    renderer.move(to: corners.last!)

    for point in corners {
      renderer.line(to: point)
    }
  }
}
```

Here's a circle, which is also a value type built out of other value types:

```swift
struct Circle: Drawable {
  var center: CGPoint
  var radius: CGFloat

  func draw(using renderer: Renderer) {
    renderer.arc(at: center, radius: radius, startAngle: 0.0, endAngle: twoPi)
  }
}
```

Now, we can build a diagram out of circles and polygons:

```swift
struct Diagram: Drawable {
  var elements = [Drawable]()

  func draw(using renderer: Renderer) {
    for element in elements {
      element.draw(using: renderer)
    }
  }
}
```

### Let's test it

Here's the test with some curiously specific values:

```swift
var circle = Circle(center: CGPoint(x: 187.5, y: 333.5), radius: 93.75)

var triangle = Polygon(corners: [
  CGPoint(x: 187.5, y: 427.25),
  CGPoint(x: 268.69, y: 286.625),
  CGPoint(x: 106.31, y: 286.625)
])

var diagram = Diagram(elements: [circle, triangle])
diagram.draw(using: Renderer())
```

The output, clearly an equilateral triangle with a circle inscribed in a circle (meant to be non-obvious/a joke):

```bash
$ ./test
Arc at (187.5, 333.5), radius: 93.75, startAngle: 0.0, endAngle: 6.28318530717959
Move to (106.310118395209, 286.625)
Line to (187.5, 427.25)
Line to (268.689881604791, 286.625)
Line to (106.310118395209, 286.625)
```

There's a good use for a text renderer: testing! We can easily see check if values change. Let's change `Renderer` to a protocol:

```swift
protocol Renderer {
  func move(to point: CGPoint)
  func line(to point: CGPoint)
  func arc(at center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat)
}

struct TestRenderer: Renderer {
  func move(to point: CGPoint) {
    print("Move to (\(point.x), \(point.y))")
  }

  func line(to point: CGPoint) {
    print("Line to (\(point.x), \(point.y))")
  }

  func arc(at center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat) {
    print("Arc at \(center), radius: \(radius), startAngle: \(startAngle), endAngle: \(endAngle)")
  }
}
```

### Making the real renderer

Now that we have our protocol defined, it's easy to turn Core Graphics into a renderer--we don't even need a new type. Recall that this makes use of *retroactive modeling*:

```swift
extension CGContext: Renderer {
  func move(to point: CGPoint) {
    CGContextMoveToPoint(self, position.x, position.y)
  }

  func line(to point: CGPoint) {
    CGContextAddLineToPoint(self, position.x, position.y)
  }

  func arc(at center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat) {
    let arc = CGPathCreateMutable()
    CGPathAddArc(arc, nil, center.x, center.y, radius, startAngle, endAngle, true)
    CGContextAddPath(self, arc)
  }
}
```

Output:

![][crustacean]

### Nesting diagrams

In our test code, what happens if we add the following line?

```swift
// ... set up circle, triangle, diagram...
diagram.elements.append(diagram)
```

We might expect our code to go into infinite recursion, but it doesn't. To learn why, see [Building Better Apps with Value Types in Swift][wwdc15414].

This code also doesn't change the display at all because it just repeats the same drawing commands twice; essentially, there's 2 diagrams, but they're directly on top of each other.

![][overlap_diagrams]

### Reflecting on `TestRenderer`

By decoupling the document model from a specific renderer, we were able to plug in a test renderer that reveals everything our code does in detail. Decoupling with protocols + generics makes our code much more testable.

This kind of testing is similar to what we might get with mocks, but it's even better. Mocks are inherently fragile; testing code needs to be coupled to the implementation details of the code under test. This fragility means mocks don't play well with Swift's strong, static type system. On the other hand, protocols use the strong type system but still give us hooks to plug in all the instrumentation we need.

### Back to our example

In our app, a bubble is just an inner circle offset around the center of the outer circle.

![][bubble]

The code looks like this. Notice how the `startAngle` and `endAngle` parameters are the same as in `Circle`:

```swift
struct Bubble: Drawable {
  // ...
  func draw(using renderer: Renderer) {
    renderer.arc(at: center, radius: radius, startAngle: 0, endAngle: twoPi)
    renderer.arc(at: highlightCenter, radius: highlightRadius, startAngle: 0, endAngle: twoPi)
  }
}

struct Circle: Drawable {
  // ...
  func draw(using renderer: Renderer) {
    renderer.arc(at: center, radius: radius, startAngle: 0.0, endAngle: twoPi)
  }
}
```

To resolve this repetition, we could start adding a new requirement to the `Renderer` protocol and then update our models to maintain conformance:

```swift
protocol Renderer {
  // move(to:), line(to:), arc(at:radius:startAngle:endAngle)
  func circle(at center: CGPoint, radius: CGFloat)
}

extension TestRenderer {
  func circle(at center: CGPoint, radius: CGFloat) {
    arc(at: center, radius: radius, startAngle: 0, endAngle: twoPi)
  }
}

extension CGContext {
  func circle(at center: CGPoint, radius: CGFloat) {
    arc(at: center, radius: radius, startAngle: 0, endAngle: twoPi)
  }
}
```

However, `TestRenderer` and `CGContext` also have duplicate implementations. We still have repeated code! 

## Protocol Extensions
Instead of duplicating code, we can use a *protocol extension* to provide a default implementation of `circle(at:radius:)` for both `TestRenderer` and `CGContext`, as well as any future renderers:

```swift
extension Renderer {
  func circle(at center: CGPoint, radius: CGFloat) {
    arc(at: center, radius: radius, startAngle: 0, endAngle: twoPi)
  }
}
```

Protocols define requirements which can be immediately fulfilled in extensions. This creates a customization point. To see how this plays out, let's see what happens when we add another method `rectangle(at:)` to the extension that isn't part of the requirements:

```swift
extension Renderer {
  func circle(at center: CGPoint, radius: CGFloat) { ... }
  func rectangle(at edges: CGRect) { ... }
}
```

Now we can extend our `TestRenderer` to implement both of these methods and then call them:

```swift
extension TestRenderer: Renderer {
  func circle(at center: CGPoint, radius: CGFloat) { ... }
  func rectangle(at edges: CGRect) { ... }
}

let r = TestRenderer()
r.circle(at: origin, radius: 1)
r.rectangle(at: edges)
```

The result isn't surprising; `circle(at:radius:)` and `rectangle(at:)` use the implementations inside `TestRenderer` instead of the default implementations from the `Renderer` protocol extension. We would get the same result if we removed any conformance.

Let's change the context; let's assume Swift only knows it has a `Renderer`, not a `TestRenderer`:

```swift
let r: Renderer = TestRenderer()
r.circle(at: origin, radius: 1)  // Uses TestRenderer implementation.
r.rectangle(at: edges)  // Uses Renderer implementation.
```

Because `circle(at:radius:)` is a requirement inside `Renderer`, our `TestRenderer` gets the privilege of customizing it, so the `TestRenderer` implementation is called. However, `rectangle(at:)` is not a requirement; the implementation in `TestRenderer` only shadows the protocol implementation. In this context, where Swift only knows it has some kind of `Renderer`, the protocol implementation is called.

Should `rectangle(at:)` have been a requirement? Possibly; some renderers are highly likely to have a more efficient implementation for their specific use case.

Should *everything* in your protocol extension also be backed by a requirement? Not necessarily; some APIs are just not intended as customization points. Sometimes, the right choice is to use the default implementation.

## More protocol extension tricks
### Constrained Extensions

We can add extensions that are only valid under some conditions. For example, this `indexOf` function would not compile because `Generator.Element` is not necessarily `Equatable`:

```swift
extension CollectionType {
  public func indexOf(element: Generator.Element) -> Index? {
    for i in self.indices {
      // ERROR: binary operator '==' cannot be applied to two Generator.Element operands
      if self[i] == element {
        return i
      }
    }
    return nil
  }
}
```

All we need to do as add a constraint to the extension so `indexOf` is only valid when the types can be compared for equality:

```swift
extension CollectionType where Generator.Element: Equatable
```

### Retroactive Adaptation
We can do something similar for our binary search. Our previous implementation was:

```swift
protocol Ordered {
  func precedes(other: Self) -> Bool
}

func binarySearch<T: Ordered>(sortedKeys: [T], forKey k: T) -> Int { ... }
```

Unfortunately, this does not work with a simple array of integers:

```swift
// ERROR: cannot invoke 'binarySearch' with an argument list of type '([Int],forKey:Int)'
let position = binarySearch([2, 3, 5, 7], forKey: 5)
```

We could fix this by adding a conformance to `Ordered` for `Int`, `String`, and so on, but that isn't maintainable. Instead, we can provide a default implementation for any type that implements `Comparable`:

```swift
extension Ordered where Self: Comparable {
  func precedes(other: Self) -> Bool {
    return self < other
  }
}
```

Now, because `Int`, `String`, `Double`, etc. all conform to `Comparable`, we can perform binary search on an array of any `Comparable` type.

### Generic beautification

This is the signature of a fully generalized binary search that works on any element with the appropriate index and element types:

```swift
// Swift 1
func binarySearch<
  C: CollectionType where C.Index == RandomAccessIndexType, C.Generator.Element: Ordered
>(sortedKeys: C, forKey k: C.Generator.Element) -> Int { ... }

let pos = binarySearch([2, 3, 5, 7, 11, 13, 17], forKey: 5)
```

Swift 1 had lots of generic free functions like this. Swift 2 used protocol extensions to make them into methods. Notice the improvement at the call site *and* at the declaration--no more angle brackets!

```swift
// Swift 2
extension CollectionType where Index == RandomAccessIndexType, Generator.Element: Ordered {
  func binarySearch(forKey: Generator.Element) -> Int { ... }
}

let pos = [2, 3, 5, 7, 11, 13, 17].binarySearch(forKey: 5)
```

### Interface generation

Here's a minimal model of Swift's `OptionSetType` protocol. Notice the broad, set-like interface we get for free just by conforming to the protocol.

![][interface_generation]

## Returning to the diagram example
### Make all value types `Equatable`
Always make value types `Equatable`. To know why, see [Building Better Apps with Value Types in Swift][wwdc15414].

`Equatable` is easy for most types--we just compare corresponding parts for equality. Unfortunately, this doesn't work with `Diagram`:

```swift
struct Diagram: Equatable {
  var elements = [Drawable]()
  func draw(using renderer: Renderer) { ... }
}

func ==(lhs: Diagram, rhs: Diagram) -> Bool {
  // ERROR: binary operator '==' cannot be applied to two [Drawable] operands.
  return lhs.elements == rhs.elements
}
```

We try comparing individual elements, but we run into the same error inside the `contains` closure because we never defined equality for `Drawable`.

```swift
func ==(lhs: Diagram, rhs: Diagram) -> Bool {
  return lhs.elements.count == rhs.elements.count
    && !zip(lhs.elements, rhs.elements).contains {
      $0 != $1  // ERROR: binary operator '==' cannot be applied to two Drawable operands.
    }
}
```

### Should every `Drawable` be `Equatable`?
The problem is that `Equatable` has `Self` requirements, which means that `Drawable` would have `Self` requirements.

```swift
protocol Equatable {
  func ==(Self, Self) -> Bool
}
```

This would put `Drawable` in the homogeneous, statically dispatched world, but a diagram requires a heterogeneous array of `Drawables`. Otherwise, we could not put polygons and circles in the same diagram.

Therefore, we should not make `Drawable` conform to `Equatable`.

### Bridge-building
We essentially need to mandate that every `Drawable` defines an equitability function:

```swift
protocol Drawable {
  func isEqualTo(_ other: Drawable) -> Bool
  // ...
}

func ==(lhs: Diagram, rhs: Diagram) -> Bool {
  return lhs.elements.count == rhs.elements.count
    && !zip(lhs.elements, rhs.elements).contains { !$0.isEqualTo($1) }
}
```

We run into a problem because comparison requires some knowledge of `Self`, i.e. the dynamic type of the `Drawable`, because we need to handle cases where we're comparing two `Drawable` objects with different dynamic types. Does this mean it's impossible to keep `Drawable`s heterogeneous?

Fortunately, in this case, no. The benefit of equality is that there's an obvious default answer of `false` if two types are not the same. So, we can implement `isEqualTo` for all `Drawable`s when the dynamic type is `Equatable`:

```swift
extension Drawable where Self: Equatable {
  func isEqualTo(_ other: Drawable) -> Bool {
    guard let o = other as? Self else { return false }
    return self == o
  }
}
```

> From a big picture perspective, we made a deal with the implementers of `Drawable`. We said, "If you really want to handle the heterogeneous case, go ahead. Otherwise, if you just want to use the regular way we express homogeneous comparison, there's a default implementation you can use."

## When to use classes
Protocol-based design applies almost universally. However, classes do still have their place.

We *want* implicit sharing when:

- It doesn't make sense to copy or compare instances
  - Ex: We don't really want to copy a graphical window. Otherwise, we would have 2 windows where 1 is not part of our view hierarchy. In this case, it makes sense to use a reference type.

- Instance lifetime is tied to external effects
  - Compilers make/destroy entities unpredictably to perform optimizations. References, on the other hand, are stable. If we depend on an external entity, we should use a reference type.

- Instances are just "sinks"
  - Here, a "sink" means something that just receives state.
  - Ex: If we wanted to make a test renderer that accumulated all commands into a string instead of writing to the console, we would want a reference type.
  - In the example below, notice that the class is `final` and that there's no inheritance; `Renderer` is still a protocol.

```swift
final class StringRenderer: Renderer {
  var result: String
  // ...
}
```

Other times to use classes:

- Don't fight the system
  - If a framework expects you to subclass or to pass an object, do so.

- On the other hand, be circumspect
  - Nothing in software should grow too large.
  - When factoring something out of a class, consider a value type.

## Summary

- Protocols > superclasses for abstraction
- Protocol extensions = magic (almost!)
- See [Building Better Apps with Value Types in Swift][wwdc15414]!

[wwdc15414]: ../414

[inheritance_hierarchies]: WWDC15-408-inheritance_hierarchies

[customization_reuse]: WWDC15-408-customization_reuse

[problem_implicit_sharing]: WWDC15-408-problem_implicit_sharing

[crustacean]: WWDC15-408-crustacean

[overlap_diagrams]: WWDC15-408-overlap_diagrams

[bubble]: WWDC15-408-bubble

[interface_generation]: WWDC15-408-interface_generation

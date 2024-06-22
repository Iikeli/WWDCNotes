# Understanding Swift Performance

In this advanced session, find out how structs, classes, protocols, and generics are implemented in Swift. Learn about their relative costs in different dimensions of performance. See how to apply this information to speed up your code.

@Metadata {
   @TitleHeading("WWDC16")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc16/416", purpose: link, label: "Watch Video (58 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



**Take-away: To write fast Swift code, avoid paying for dynamism that you’re not taking advantage of.**

Dimensions of Performance:

1. Allocation
2. ARC
3. Method dispatch

## 1. Allocation
- Stack allocation is very fast; the cost of assigning an integer.
- Heap is more dynamic but less efficient, O(log n). Also needs to protect integrity by checking for thread safety which is lots of overhead.

### Example: Creating a point struct.
```swift
struct Point {
  var x, y: Double
  func draw() { ... }
}

let point1 = Point(x: 0, y: 0)
var point2 = point1

// Since `Point` is a struct, this creates a copy. Modifying `point1` does not change `point2`.
point2.x = 5
```
- Note that `point2 = point1` creates a copy since these are structs, which use value semantics. They are different.

### Example: Creating a point class.
```swift
class Point {
  var x, y: Double
  func draw() { ... }
}

let point1 = Point(x: 0, y: 0)
var point2 = point1

// Since `Point` is a class, this adds a new reference to the same block of memory on the heap.
// The value of `x` is `5` for both references after this assignment.
point2.x = 5
```

### Example to make the background balloon in a chat application. This function is called a lot, so how can we make it fast?
```swift
enum Color {
  case blue, green, gray
}

enum Orientation {
  case left, right
}

enum Tail {
  case none, tail, bubble
}

// Never construct something more than once; if you need it again, it’s already here.
var cache = [String: UIImage]()

func makeBalloon(_ color: Color, orientation: Orientation, tail: Tail) -> UIImage {
  // Add
  let key = “\(color):\(orientation):\(tail)”
  if let image = cache[key] {
    return image
  }

  // If not cached, make a new balloon...
}
```

What’s bad?

- A string is almost limitless in terms of the values it can represent. It’s not a strong key. You could put the name of your dog as a key.
- A string stores the contents of its characters indirectly on the heap. Even if you get a cache hit, you’re still incurring a heap allocation when you construct the key.

How can you make this better?

- Represent the key as a struct of its 3 attributes. They are first class types and can be used as a key in the dictionary.
- It’s a lot safer and a lot faster.

```swift
enum Color {
  case blue, green, gray
}

enum Orientation {
  case left, right
}

enum Tail {
  case none, tail, bubble
}

struct Attributes: Hashable {
  var color: Color
  var orientation: Orientation
  var tail: Tail
}

var cache = [Attribute: UIImage]()

func makeBalloon(_ color: Color, orientation: Orientation, tail: Tail) -> UIImage {
  let key = Attributes(color: color, orientation: orientation, tail: tail)
  if let image = cache[key] {
    return image
  }

  // If not cached, make a new balloon...
}
```

## 2. ARC
- To know when it's safe to deallocate an instance on the heap, Swift uses reference counting. If no one is pointing to it, Swift deallocates it.
- **ARC is more expensive than just incrementing/decrementing**.
- There's a lot of indirection and thread safety checks. Need to do this atomically, and the cost adds up.

Back to the point class example. This is what's actually happening behind the scenes.


```swift
class Point {
  var refCount: Int
  var x, y: Double
  func draw() { ... }
}

// Initialization assigns `refCount` to `1`.
let point1 = Point(x: 0, y: 0)

var point2 = point1
retain(point2)  // Increment `refCount`.
point2.x = 5

// Use `point1`.
release(point1)  // Decrement `refCount`.

// Use `point2`.
release(point2)
```

What about the point struct? There's no reference counting overhead; the code is exactly the same.

However, let's consider a more complicated struct:
```swift
struct Label {
  var text: String  // Remember, `String` stores its characters on the heap.
  var font: UIFont  // `UIFont` is a class. This is also tracked on the heap.
  func draw() { ... }
}

// `Label` is declared on the stack, but its attributes have 2 pointers to the heap.
let label1 = Label(text: "hi", font: font)

// Since structs are value types and are copied, this means there are 2 *more* pointers
// to the same String and UIFont memory on the heap, causing 2x the overhead!
let label2 = label1
```

Behind the scenes, this code is:
```swift
struct Label {
  var text: String
  var font: UIFont
  func draw() { ... }
}

let label1 = Label(text: "hi", font: font)
let label2 = label1

retain(label2.text._storage)
retain(label2.font)

// Use `label1`...

release(label1.text._storage)
release(label1.font)

// Use `label2`...

release(label2.text._storage)
release(label2.font)
```

The take-away: structs are only advantageous in terms of ARC if they have 0 or 1 reference types as attributes.

- Instead of `uuid: String`, use Foundation's built-in `UUID` type.
- Instead of strings, use enums.

# 3. Method Dispatch
There's static dispatch and dynamic dispatch.

If Swift can tell which method to execute at compile time, then that's called static dispatch and you can jump straight into that code at run-time. The compiler can also optimize it aggressively into the surrounding code, either by inlining (literally copying and pasting the function body where the call is) or other tricks. This helps avoid new stack frames, etc. You can collapse a chain of static dispatches into the code that you actually want.

Otherwise, you use dynamic dispatch, which has the overhead of figuring out which method to call before you even get to the code and you give up compiler optimizations which is the main drawback.

Why use dynamic dispatch? Inheritance-based polymorphism gives you more flexibility in your code, but you give up static dispatching.

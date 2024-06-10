# Embrace Swift generics

Generics are a fundamental tool for writing abstract code in Swift. Learn how you can identify opportunities for abstraction as your code evolves, evaluate strategies for writing one piece of code with many behaviors, and discover language features in Swift 5.7 that can help you make generic code easier to write and understand.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110352", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(Jeehut)
      @GitHubUser(zntfdr)
   }
}



> For the purpose of this talk, imagine to have an `Animal` protocol

- Generics are here to abstract away the details of a specific type
- if you find yourself writing overloads with repetitive implementations, it might be a sign to that you need to generalize
- Start with concrete types, generalize when needed

## Polymorphism

- ability of abstract code to behave differently for different concrete types
- allows one piece of code to have many behaviors depending on how the code is used

Different forms:

- <kbd>ad-hoc polymorphism</kbd>
  - the same function call can mean different things depending on the argument type
  - Swift function overloading

- <kbd>subtype polymorphism</kbd>
  - code operating on a supertype can have different behavior based on the specific subtype the code is using at runtime
  - Subclassing in Swift, where a class `override`s a method of their superclass

- <kbd>parametric polymorphism</kbd> - achieved using generics

- Generic code uses type parameters to allow writing one piece of code that works with different types, and concrete types themselves are used as arguments

## `protocol`

- interface that represents capabilities
- separates ideas from implementation details
- abstraction tool that describes the functionality of conforming types
- separates ideas from implementation details
- each capability maps to a protocol requirement
- the name of the protocol should represent the category of types we're describing

### `associatedtype` 

- serves as a placeholder for a concrete type
- associated types depend on the specific type that conforms to the protocol 

## Opaque type vs. underlying type

- <kbd>Opaque type</kbd> - abstract type that represents a placeholder for a specific concrete type
- <kbd>underlying type</kbd> - specific concrete type that is substituted in the opaque type

- For values with opaque type, the underlying type is fixed for the scope of the value
- generic code using the value is guaranteed to get the same underlying type each time the value is accessed
- opaque type can be used both for inputs and outputs

Opaque types in Swift:

- `some Animal`
- `<T: Animal>`

## Tips on writing generic interfaces (protocols or not)

### `some`

Swift 5.6 and earlier:

```swift
func feed<A>(_ animal: A) where A: Animal

// 👆🏻👇🏻 Equivalents

func feed<A: Animal>(_ animal: A)
```

Swift 5.7 and later:

```swift
func feed(_ animal: some Animal)
```

- we can express an abstract type in terms of the protocol conformance by writing `some xxx`
- all three declarations above are identical, but the newer one is much easier to read and understand
- the `some` keyword can be used in parameter and result types
- with `some`, there is a specific underlying type that cannot vary

### `any`

- if we need to express an arbitrary type a protocol, for example to store multiple into an array, we can use `any`
- with `any`, the underlying type can vary at runtime
- `any xxx` is a single static type that has the capability to store any concrete `X`-conforming type dynamically, which allows us to use subtype polymorphism with value types
- to allow for `any` required flexible storage, the `any X` type has a special representation in memory
  - you can think of this representation as a fixed box: 
    - if the concrete type can fit within the box, it will be stored there
    - the box will store a pointer to the instance otherwise

- the static type `any X` that can dynamically store any concrete `X` type is called an <kbd>existential type</kbd>
- the strategy of using the same representation for different concrete types is called <kbd>type erasure</kbd>
- the concrete type is said to be _erased_ at compile time, and the concrete type is only known at runtime
- type erasure eliminates the type-level distinction between different `X`-conforming instances, which allows us to use values with different dynamic types interchangeably as the same static type

## `some` vs. `any`

- the compiler can convert an instance of `any X` to `some x` by unboxing the underlying value and passing it directly to the `some X` parameter
  - new in Swift 5.7

| `some` | `any` |
| --- | --- |
| holds a fixed concrete type | holds an arbitrary concrete type |
| guarantees type relationships | erases type relationships |

Guidelines:

- use `some` by default
- change to `any` when you need to store arbitrary values (arbitrary concrete types instances)
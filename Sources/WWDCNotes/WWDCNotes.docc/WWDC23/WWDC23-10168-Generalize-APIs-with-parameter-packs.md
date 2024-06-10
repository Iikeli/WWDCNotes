# Generalize APIs with parameter packs

Swift parameter packs are a powerful tool to expand what is possible in your generic code while also enabling you to simplify common generic patterns. We’ll show you how to abstract over types as well as the number of arguments in generic code and simplify common generic patterns to avoid overloads.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10168", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(rogerluan)
   }
}



- This is an advanced talk. Deep understanding of Swift Generics and Variadics is required.
- Feature introduced in Swift 5.9, Xcode 15.

## What Parameter Packs Solves

You can use variadics to pass a variable number of arguments to a function, but all arguments must be of the same type.

Problem 1: you can't pass a variable number of arguments of different types without using type erasure.

Problem 2: let's say you want the return value of said function to be a tuple with the number of elements depending on the number of arguments that was passed in. You can't do that either.

What we lack with the generics system and variadic parameters is the ability to both preserve type information and vary the number of arguments. The only way to do this today is with overloading, which forces you to choose an upper bound of the number of arguments you support.

tl;dr: remember `zip3(…)`, `zip4(…)`, `zip5(…)`, `zip6(…)`…? Parameter packs solve that.

## How to Read Parameter Packs (Introducing The New Syntax)

A type pack is a list of types. A value pack is a list of values.

```swift
func query<each Payload> // Example of a type parameter pack
```

Naming convention: use the singular form after the keyword `each`, e.g. `each Payload`, instead of `each Payloads`.

> For the remaining of this section, I'll paraphrase the presenter and their code examples, because the session video was spot on. If you have the chance, watch the session video from 06:45 to 08:27 to visualize the animated code examples, it's worth it.

Generic code that uses parameter packs can operate on each Payload individually using repetition patterns. A repetition pattern is expressed using the 'repeat' keyword, followed by a type called the pattern type. The pattern will contain one or more references to pack elements. 'repeat' indicates that the pattern type will be repeated for every element in the given argument pack. 'each' acts as a placeholder that is replaced with individual pack elements at every iteration:

```swift
repeat Request<each Payload>
```

Let's see how this replacement works with a concrete type pack containing Bool, Int, and String. The pattern will be repeated three times and the placeholder 'each Payload' is replaced with the concrete type in the pack during each repetition:

```swift

Request<each Payload>, Request<each Payload>, Request<each Payload>

        Bool                   Int                    String
```

The result is a comma-separated list of types: Request of Bool, Request of Int, and Request of String:

```swift
Request<Bool>, Request<Int>, Request<String>
```

Because repetition patterns produce comma-separated lists of types, they can only be used in positions that naturally accept comma-separated lists. This includes types wrapped in parentheses, which are either a tuple type or a single type:

```swift
(repeat Request<each Payload>)
(Request<Bool>, Request<Int>, Request<String>)
```

Repetition patterns can be used in generic argument lists:

```swift
Generic<repeat Request<each Payload>>
Generic<Request<Bool>, Request<Int>, Request<String>>
```

They can also be used in function parameter lists:

```swift
(_ item: repeat Request<each Payload>) -> Bool
```

Basically, when using this with function parameter, it's the equivalent of variadic parameters, but with type information preserved.

Thus, this mess:

```swift
func query<Payload>(
  _ item: Request<Payload>
) -> Payload

func query<Payload1, Payload2>(
  _ item1: Request<Payload1>,
  _ item2: Request<Payload2>
) -> (Payload1, Payload2)

func query<Payload1, Payload2, Payload3>(
  _ item1: Request<Payload1>,
  _ item2: Request<Payload2>,
  _ item3: Request<Payload3>
) -> (Payload1, Payload2, Payload3)

func query<Payload1, Payload2, Payload3, Payload4>(
  _ item1: Request<Payload1>,
  _ item2: Request<Payload2>,
  _ item3: Request<Payload3>,
  _ item4: Request<Payload4>
) -> (Payload1, Payload2, Payload3, Payload4)
```

Becomes this one-liner, for any number of arguments:

```swift
func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload)
```

Note that the number of elements returned in the return value will match the number of elements passed in the argument pack.

You can make your generic `Payload` conform to protocols like usual, e.g.:

```swift
func query<each Payload: Equatable>(_ item: repeat Request<each Payload>) -> (repeat each Payload)

// Or

func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload) where repeat each Payload: Equatable
```

Implementing a function with a minimum of 1 argument:

```swift
func query<FirstPayload, each Payload>(
  _ first: Request<FirstPayload>, _ item: repeat Request<each Payload>
) -> (FirstPayload, repeat each Payload)
  where FirstPayload: Equatable, repeat each Payload: Equatable
```

## Using Parameter Packs

Use this syntax to iterate over the elements of a pack:

```swift
struct Request<Payload> {
  func evaluate() -> Payload
}

func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload) {
  return (repeat (each item).evaluate())
}
```

The parenthesis around the `repeat (each item).evaluate()` make the return value return a tuple. Note that it's clever enough so that, if the parameter pack only has one element, it will return a single value instead of a tuple.

# Related Sessions

- [Embrace Swift Generics - WWDC22](https://wwdcnotes.com/notes/wwdc22/110352/)
- [Design protocol interfaces in Swift - WWDC22](https://wwdcnotes.com/notes/wwdc22/110353/)

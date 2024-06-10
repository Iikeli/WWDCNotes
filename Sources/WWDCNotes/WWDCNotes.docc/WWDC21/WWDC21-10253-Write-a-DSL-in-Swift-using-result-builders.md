# Write a DSL in Swift using result builders

Some problems are easier to solve by creating a customized programming language, or “domain-specific language.” While creating a DSL traditionally requires writing your own compiler, you can instead use result builders with Swift 5.4 to make your code both easier to read and maintain. We’ll take you through best practices for designing a custom language for Swift: Learn about result builders and trailing closure arguments, explore modifier-style methods and why they work well, and discover how you can extend Swift’s normal language rules to turn Swift into a DSL.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10253", purpose: link, label: "Watch Video (46 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample Code](https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui)

## What's a DSL

**D**omain-**S**pecific **L**anguage

- Programming language which builds in an assumption about the problem space
- Because the language is designed with a particular kind of work in mind, it can have special features which make that kind of work easier to do
- Your code only specifies the custom parts; language adds implicit behavior
- Often declarative (think like SwiftUI)
- <kbd>Standalone DSL</kbd>
  - tranditional way to create DSL
  - you'd design the entire language from scratch and write an interpreter/compiler for it
- <kbd>Embedded DSL</kbd>
  - you use the built-in features of a host language (e.g., Swift) to add the DSL’s implicit behavior to some parts of your code, effectively modifying the host language into one tailored for your domain (e.g., SwiftUI)
  - easier to implement
  - the parts written in the DSL look like normal code to the rest of the app, so you have a much easier time interoperating
  - lets you use existing tool for the host language (e.g., debuggers and editors for Swift)
  - easier to learn

Swift features are used together to build the SwiftUI DSL:

- Property wrappers - These let clients declare variables that are tied to DSL behavior
- Trailing closure arguments - These let the DSL provide functions or initializers that read almost like custom syntax that’s been added to the language
- Result builders - These collect the values computed in your DSL’s code into a return value so you can process them
- Modifier-style methods - These are methods that return a wrapped or modified version of the value they’re called on

```swift
struct FavoriteSmoothies: View {
  @EnvironmentObject // 👈🏻 Property wrappers
  private var model: FrutaModel

  var body: some View {
    SmoothieList(smoothies: model.favoriteSmoothies)
      .overlay(
        Group { // 👈🏻 Trailing closure arguments
          if model.favoriteSmoothies.isEmpty { // 👈🏻 Result builders
            Text("Add some smoothies!")
              .foregroundColor(.secondary)
              .frame(maxWidth: .infinity, maxHeight: .infinity) 
          }
        }
      )
      .navigationTitle("Favorites") // 👈🏻 Modifier-style methods
  }
}
```

## How result builders work

- Help to implement your DSL's domain-specific assumed behavior
- Applied to a function, method, getter, or closure
- Wrap statements in implicit method calls so you can use their results
- Take over function's return value
  - When you apply a result builder to a function body, Swift inserts various calls to static methods on the result builder. These end up capturing the results of statements that would otherwise have been discarded. So where Swift would normally ignore a return value, it instead gets passed to the result builder. These calls ultimately compute a value which is returned from the function body.
- Compile time feature

SwiftUI example:

![][swiftuiExampleGif]

In short, we go from

```swift
VStack {
  Text("Title").font(.title)
  Text("Contents")
}
```

to:

```swift
VStack.init(content: {
  let v0 = Text("Title").font(.title)
  let v1 = Text("Contents")
  return ViewBuilder.buildBlock(v0, v1)
})
```

Result builders limitations:

- Syntax in a result builder function is the same as the host language
- Looks for names in the normal places
- Result builders disable some language keywords (mostly interrupt controls like `try` `catch`, that don't fit well into statement results)
  - some keywords are enabled conditionallv (depending whether the result builder provides extra methods that are used to implement them)

- If a keyword is permitted, it will work as usual - Result builders just capture the statement results that would otherwise have been thrown away, nothing more

## Writing a result builder

1. define an enum `@resultBuilder`:

```swift
@resultBuilder
enum SmoothieArrayBuilder {
}
```

This type is a container for result builders static methods, hence declaring it as a caseless enum makes sense

2. all result builders require a `buildBlock(_:)` method:

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildBlock(_ components: Smoothie...) -> [Smoothie] {
    components
  }
}
```

Each statement in your result builder body gets assigned to a variable, and then all variables are all passed to `buildBlock(_:)` for you to use

2. (optional) if you need to support if statements, define the associated `buildOptional(_:)` method:

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  /// This is called once for the if closure and once when the if closure is not called (`buildOptional(nil)` will be called).
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    component ?? []
  }
  
  static func buildBlock(_ components: Smoothie...) -> [Smoothie] {
    components
  }
}
```

However, we now need a `buildBlock(_:)` definition that accepts both `Smoothie` and `[Smoothie]`, two ways to fix this:

- Make the parameter type match both `Smoothie` and `[Smoothie]` (this is how SwiftUI solves this)
- Convert Smoothies returned by statements into `[Smoothie]`, we can do this via `buildExpression(_:)`

`buildExpression(_:)` passes each bare expression to that method before it captures it into a variable (that is then passed to `buildBlock(_:)`.

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    component ?? []
  }
  
  static func buildBlock(_ components: [Smoothie]...) -> [Smoothie] {
    components.flatMap { $0 }
  }
  
  static func buildExpression(_ expression: Smoothie) -> [Smoothie] {
    [expression]
  }
}
```

3. (optional) add support for if-else, if let, switch via `buildEither(first:)` and `buildEither(second:)`

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildEither(first component: [Smoothie]) -> [Smoothie] {
    component
  }
  
  static func buildEither(second component: [Smoothie]) -> [Smoothie] {
    component
  }
  
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    component ?? []
  }
  
  static func buildBlock(_ components: [Smoothie]...) -> [Smoothie] {
    components.flatMap { $0 }
  }
  
  static func buildExpression(_ expression: Smoothie) -> [Smoothie] {
    [expression]
  }
}
```

[swiftuiExampleGif]: swiftui-example.gif
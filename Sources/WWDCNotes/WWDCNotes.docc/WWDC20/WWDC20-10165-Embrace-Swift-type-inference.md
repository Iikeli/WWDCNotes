# Embrace Swift type inference

Swift uses type inference to help you write clean, concise code without compromising type safety. We’ll show you how the compiler seeks out clues in your code to solve the type inference puzzle. Discover what happens when the compiler can't come to a solution, and find out how Xcode 12 integrates error tracking to help you understand and fix mistakes at compile time.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10165", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



Topics covered:

1. Leveraging type inference
2. How type inference works in the compiler
3. Using Swift and Xcode to fix compiler errors

## Review: What is Type Inference?
We can omit type annotations and other verbose details in our source code if the compiler can figure out those details from context.

If we write:

```swift
let x = ""
```

Then the compiler infers `x` to be of type `String` because the value is a string literal. This can be explicitly written in one of two ways:

```swift
let x: String = ""
// or
let x = "" as String
```

This is a small example, but type inference can get very powerful in large projects. For example, SwiftUI projects are composed of small, reusable views.

## 1. Leveraging type inference at the call site.
The sample project here is Fruta, a smoothie app. We want to add a search feature so that users can search for smoothies.

![][search_smoothie]

This is how the current `SmoothieList` view is implemented. It maps an array of `Smoothie` to a list of `SmoothieRowView`.

```swift
import SwiftUI

struct SmoothieList: View {
  var smoothies: [Smoothie]

  var body: some View {
    List(smoothies) { smoothie in
      SmoothieRowView(smoothie: smoothie)
    }
  }
}
```

To add search functionality, we need to filter the array of `Smoothie` using a `String` that the user searched for. We add that variable to `SmoothieList`:

```swift
@State var searchPhrase = ""
```

We also need to use a custom list view called `FilteredList`, which only shows objects that match a certain condition. So, we replace:

```swift
List(smoothies) { smoothie in ... }
```

with

```swift
FilteredList(
  smoothies,
  filterBy: \.title
  isIncluded: { title in title.hasSubstring(searchPhrase) }
) { smoothie in ... }
```

In this case, smoothies will only be included if their `title` field has the search phrase.

**This call to the `FilteredList` initializer leans heavily on type inference.** Let's look at the declaration of `FilteredList` and its initializer.

`FilteredList` is a general-purpose view, so it should work with any type of object. This is where generics come in handy.

```swift
import SwiftUI

public struct FilteredList<Element, FilterKey, RowContent>: View {
  public init() { ... }

  public var body: some View { ... }
}
```

`Element`, `FilterKey`, and `RowContent` are all placeholders to be replaced with actual types at the call site. These actual types (formally called "concrete types") are either specified at the call site or inferred by the compiler. In this case, `Element` is a placeholder for the array element type, `FilterKey` is a placeholder for the property on which to filter, and `RowContent` is a placeholder for the type of view to show in each row of the list.

Now, let's use these in the initializer. Any time you can use a generic in the initializer, you should do so, because that means less verbosity at the call site.

```swift
public init(
  _ data: [Element],
  filterBy key: KeyPath<Element, FilterKey>,
  isIncluded: @escaping (FilterKey) -> Bool,  // 1
  @ViewBuilder rowContent: @escaping (Element) -> RowContent  // 2
)
```

1. `isIncluded` is marked as `@escaping` because it will need to be stored in a property, so it must exist outside the lifetime of the initializer.
2. This is a function to map an element to a view. It's also escaping because it needs to be stored.

Also, the `@ViewBuilder` enables SwiftUI DSL syntax. We can define several child views by listing them out in the body of the closure. The `ViewBuilder` will collect the child views into a tuple for the parent to work with.

Here's an image of the definition and call site of `FilteredList` to show how the call site leans on type inference. Notice how clean the call site is. There are no explicit type annotations, but the compiler still has all the information it needs.

```swift
// FilteredList. swift 
public struct FilteredList<Element, FilterKey, RowContent> { 
  public init(
    _ data: [Element], 
    filterBy key: KeyPath<Element, FilterKey>, 
    isIncluded: @escaping (FilterKey) -> Bool, 
    @ViewBuilder rowContent: @escaping (Element) -> RowContent
  )
} 


FilteredList(
  smoothies,
  filterBy: \.title,
  isIncluded: { title in title.hasSubstring(searchPhrase) }
) { smoothie in 
  SmoothieRowView(smoothie: smoothie) 
}
```

If all the argument types were explicitly specified in the code, the call site would look like this:

```swift
FilteredList<Element, FilterKey, RowContent>(
  smoothies as [Element],
  filterBy: \Element.title as KeyPath<Element, FilterKey>,
  isIncluded: { (title: FilterKey) -> Bool in title.hasSubstring(searchPhrase) },
) { (smoothie: Element) -> RowContent in
  SmoothieRowView(smoothie: smoothie)
}
```

Type inference helps us write source code faster because we don't need to explicitly specify all these types in our code.

## 2. How type inference works in the compiler
Think of type inference like a puzzle. The inference algorithm "fills in" the puzzle using clues from the source code. Filling in just one piece can also uncover more clues about the remaining pieces.

We're going to type-infer the call site of `FilteredList` just like the compiler.

We start with `Element`. The call site has:

```swift
smoothies as [Element]
```

We know `smoothies` is of type `[Smoothie]`, so that means that `Element = Smoothie`. Let's do a find + replace on the verbose version of the call site:

```swift
FilteredList<Smoothie, FilterKey, RowContent>(
  smoothies as [Smoothie],
  filterBy: \Smoothie.title as KeyPath<Smoothie, FilterKey>,
  isIncluded: { (title: FilterKey) -> Bool in title.hasSubstring(searchPhrase) },
) { (smoothie: Smoothie) -> RowContent in
  SmoothieRowView(smoothie: smoothie)
}
```

Now, filling in a concrete type for `Element` gave a clue about the concrete type of `FilterKey` because now we know that `KeyPath` literal is referring to `Smoothie.title`. We know that `Smoothie.title` is a `String`, therefore `FilterKey` is also a `String`. Let's do another find + replace.

```swift
FilteredList<Smoothie, String, RowContent>(
  smoothies as [Smoothie],
  filterBy: \Smoothie.title as KeyPath<Smoothie, String>,
  isIncluded: { (title: String) -> Bool in title.hasSubstring(searchPhrase) },
) { (smoothie: Smoothie) -> RowContent in
  SmoothieRowView(smoothie: smoothie)
}
```

The last piece of the puzzle is `RowContent`, which is the return type of the trailing `ViewBuilder` closure. Since this closure only has 1 view in the body, the `ViewBuilder` will return the same type as the child view, `SmoothieRowView`. Find + replace again:

```swift
FilteredList<Smoothie, String, SmoothieRowView>(
  smoothies as [Smoothie],
  filterBy: \Smoothie.title as KeyPath<Smoothie, String>,
  isIncluded: { (title: String) -> Bool in title.hasSubstring(searchPhrase) },
) { (smoothie: Smoothie) -> SmoothieRowView in
  SmoothieRowView(smoothie: smoothie)
}
```

We've solved the last piece of the puzzle. This is the same strategy that the compiler uses with our code. Each step of the algorithm uncovers more clues for the next step.

However, it's possible for one of the clues to cause the compiler to fill in a concrete type that doesn't fit in with the rest of the puzzle. If one of the pieces doesn't fit and the puzzle can't be solved, there's an error in the source code.

### Solving the puzzle in the presence of source code errors
Let's rewind to when the compiler found the concrete type for `FilterKey`:

```swift
FilteredList<Smoothie, FilterKey, RowContent>(
  smoothies as [Smoothie],
  filterBy: \Smoothie.title as KeyPath<Smoothie, FilterKey>,
  isIncluded: { (title: FilterKey) -> Bool in title.hasSubstring(searchPhrase) },
) { (smoothie: Smoothie) -> RowContent in
  SmoothieRowView(smoothie: smoothie)
}
```

In the previous step, the compiler inferred `Smoothie` as the `KeyPath` base type. It used this information to figure out the concrete type for `FilterKey` by looking up the type of `Smoothie.title`.

What if we had incorrectly passed `Smoothie.isPopular: Bool` instead of `Smoothie.title: String`? The compiler would have tried to infer the type of `FilterKey` to be `Bool`. It would continue to fill in the other `FilterKey` placeholders with that same incorrect type. Eventually, it would have tried to make sense of the following line of code:

```swift
isIncluded: { (title: Bool) -> Bool in title.hasSubstring(searchPhrase) }
```

`Bool` does not have a property `hasSubstring` and the compiler would report an error.

## 3. Using Swift and Xcode to fix compiler errors
The Swift compiler is designed to catch mistakes by integrating error tracking into the type inference algorithm to use later on in error messages.

### Integrated error tracking
During type inference, the compiler:

1. Records information about errors in source code
2. Attempts to fix errors using heuristics so it can continue the type inference algorithm
3. Provides actionable error messages based on collected information

Integrated error tracking was introduced in Swift 5.2 and Xcode 11.4. In Swift 5.3 and Xcode 12, the compiler uses this new strategy for all error messages and expressions.

### Using Xcode to fix errors in Swift code
Before writing any code, open up Xcode > Behaviors > Edit Behaviors. Add a behavior to automatically show the issue navigator when the build fails:

![][fail_behavior]

Now, Xcode will show all the errors across the project each time it fails to build.

The current scene shows the implementation of `SmoothieList` and its preview. `FilteredList` has already been added to the project.

![][smoothie_preview]

Before replacing `List` with `FilteredList`, we need to add a search field above the list as a `TextField`.

![][textfield_vstack]

After attempting to build, there is a compiler error on the line of code that was just added.

![][compiler_error_line]

We can expand the error by clicking on it.

![][compiler_error_expanded]

The error tells us that we provided a `String`, but the `TextField` expects a different type. We made the mistake of passing the value, not the binding. The Swift compiler was able to figure out that the binding *does* have a compatible type, and it provided a fix-it to refer to the binding with `$`.

Next, we want to replace `List` with `FilteredList` as in the code examples above. We attempt to build and get another error:

![][filteredlist_error]

The error tells us that `Smoothie` needs to conform to `Identifiable` in order to work with `FilteredList`. That might be confusing because `Identifiable` isn't written anywhere in this code. However, in the left, there is a compiler note in gray attached to the error saying "Where 'Element' = 'Smoothie'". This note is a "breadcrumb" from the compiler so we know what it was doing.

To view the note side-by-side with the error, we can close the preview canvas with the "CMD+Enter" shortcut. Then, we can hold down "Option+Shift" while clicking on the gray compiler note. This opens the Destination Chooser.

![][destination_chooser]

We can move our cursor over to the right and hit "Enter" to open the source of the error in a new editor on the right.

![][side_by_side_error]

Looking at the full declaration, we can see that `Element` must conform to `Identifiable`. Because the compiler inferred `Smoothie` as the concrete type of `Element`, `Smoothie` must also conform to `Identifiable`.

![][conformance]

All we need to do is jump to the definition of `Smoothie` and add a conformance to `Identifiable`. `Smoothie` already has a property called `id`, so it is eligible to conform.

![][add_conformance]

We attempt to build again, and it succeeds this time.

## Wrap up

- SwiftUI code relies on type inference for reusable views
- Type inference fills in incidental details using clues from source code
- Integrated error tracking leaves breadcrumbs for error messages

To learn more about integrated error tracking, read the [blog post on Swift's new diagnostic architecture](https://swift.org/blog/new-diagnostic-arch-overview/).

To learn more about generics, watch [WWDC 2018's video on Swift Generics](../../wwdc18/406).

[search_smoothie]: WWDC20-10165-search_smoothie
[fail_behavior]: WWDC20-10165-fail_behavior
[smoothie_preview]: WWDC20-10165-smoothie_preview
[textfield_vstack]: WWDC20-10165-textfield_vstack
[compiler_error_line]: WWDC20-10165-compiler_error_line
[compiler_error_expanded]: WWDC20-10165-compiler_error_expanded
[filteredlist_error]: WWDC20-10165-filteredlist_error
[destination_chooser]: WWDC20-10165-destination_chooser
[side_by_side_error]: WWDC20-10165-side_by_side_error
[conformance]: WWDC20-10165-conformance
[add_conformance]: WWDC20-10165-add_conformance
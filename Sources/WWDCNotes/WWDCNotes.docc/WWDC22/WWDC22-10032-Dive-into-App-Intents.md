# Dive into App Intents

Learn how you can make your app more discoverable and increase app engagement when you use the App Intents framework. We'll take you through the powerful capabilities of this Swift framework, explore the differences between App Intents and SiriKit Intents, and show you how you can expose your app's functionality to the system. We'll also share how you can build entities and queries to create rich App Shortcuts experiences.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10032", purpose: link, label: "Watch Video (30 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## Introduction

- [new framework][appintents]
- three key components: [`Intents`][app-intents], [`Entities`][app-entities], [`App Shortcuts`][app-shortcuts]
- With App Shortcuts, everyone can use them via voice from Siri, they also appear in Spotlight
- Intents allow to build focus filters
  - e.g., Calendar.app only shows work calendar when in work mode

- Users can invent entirely new features and capabilities

## Intents and parameters

- [Intent][app-intents] is a single piece of app functionality
  - e.g., “make a new calendar event”, “open a particular screen”

- Performed manually or automatically
- either returns a [`IntentResult`][IntentResult] or throws an `Error` (possibly a [`IntentError`][IntentError]?)
- three key pieces: 
  - the intent metadata - e.g., its title and description, shown to the user
  - the intent parameters - all values can be customized by the user
  - the intent [`perform()`][appintent/perform()] method - which is triggered when the user wants the intent to execute

- Example:

```swift
struct OpenCurrentlyReading: AppIntent {
  static var title: LocalizedStringResource = "Open Currently Reading"

  @MainActor // 👈🏻 ensure it's executed in the main thread
  func perform() async throws -> some PerformResult { // 👈🏻
    Navigator.shared.openShelf(.currentlyReading)
    return .finished
  }

  static var openAppWhenRun: Bool = true
}
``` 

- This simple `OpenCurrentlyReading` definition automatically makes our app intent available in the following places:
  - Menu Bar
  - Share Extensions 
  - Terminal
  - AppleScript
  - Home Screen
  - Suggestions
  - Lock Screen
  - Shortcuts Widgets
  - Quick Actions
  - Voice (Siri)
  - Apple Watch
  - HomePod
  - Automations
  - Shortcuts App
  - Keyboard
  - Spotlight

- make your custom types conform to [`AppEnum`][AppEnum] to express that a custom type has a predefined, static set of valid values to display
  - can be used for types that have a known set of valid values

```swift
public enum Shelf: String, AppEnum {
  case currentlyReading
  case wantToRead
  case read

  static var typeDisplayName: LocalizedStringResource = "Shelf"

  static var caseDisplayRepresentations: [Shelf: DisplayRepresentation] = [
    .currentlyReading: "Currently Reading",
    .wantToRead: "Want to Read",
    .read: "Read",
  ]
}
```

- Use [`@Parameter(title:)`][IntentParameter] to define your intent parameters

```swift
struct OpenShelf: AppIntent {
  static var title: LocalizedStringResource = "Open Shelf"

  @Parameter(title: "Shelf") // 👈🏻
  var shelf: Shelf

  @MainActor
  func perform() async throws -> some PerformResult {
    Navigator.shared.openShelf(shelf)
    return .finished
  }

  static var parameterSummary: some ParameterSummary {
    Summary("Open \(\.$shelf)")
  }

  static var openAppWhenRun: Bool = true
}
```
- Supported parameters types:
  - Decimal
  - Person
  - Location
  - URL
  - Integer
  - File
  - Payment Method
  - Rich Text
  - Boolean
  - Measurement
  - Enumeration
  - String
  - Date
  - Duration
  - [App Entity][app-entities]
  - Currency

- Always provide a parameter `summary` for every intent you create, supports `when`, `otherwise`, `switch` and `case` APIs
- use static property [`openAppWhenRun`][openAppWhenRun] to open app on running

## Entities, queries, and results

- Entity contains identifier, display of representation and type name
- Any struct can conform to [`AppEntity`][AppEntity]:

```swift
struct BookEntity: AppEntity, Identifiable {
  var id: UUID // 👈🏻 UUID is a good identifier type
  var title: String
  
  var displayRepresentation: DisplayRepresentation {
    DisplayRepresentation(title: LocalizedStringResource(stringLiteral: title))
  }

  static var typeDisplayName: LocalizedStringResource = "Book"
  
  static var defaultQuery = BookQuery()
}
```

### [Entity queries][entity-queries]

- help the system find the entities your app defines and use them to resolve parameters.
- `StringQuery` and `PropertyQuery` to look up entities
- all queries support suggestions
- conform to `EntityQuery` on structs for queries
- hook them up by adding `defaultQuery` to entity
- conform to `EntityStringQuery` for string search, e.g. books
- conform error to `CustomLocalizedStringResourceConvertible`
- provide `ReturnsValue` if you want your shortcut return a result
- adopt [`OpensIntent`][openentityintent] protocol in return type to show open button so users can select if app is opened or not

```swift
struct BookQuery: EntityQuery {
  func entities(for identifiers: [UUID]) async throws -> [BookEntity] {
    identifiers.compactMap { identifier in
      Database.shared.book(for: identifier)
    }
  }
}
```

### Properties, finding and filtering

- Property queries find entities on the properties within entity
- Three steps:
  1. Declare query properties
  2. Declare sorting options
  3. Implement `entities(matching:)` to run the search

- [Supported query properties][property-comparators]
  - examples: 
    - <kbd>less than</kbd> and <kbd>greater than</kbd> for `Date`s
    - <kbd>contains</kbd> and <kbd>equal to</kbd> for `String`s

- Conform to [`EntityPropertyQuery`][EntityPropertyQuery] with your comparators:

```swift
struct BookQuery: EntityPropertyQuery {
  static var sortingOptions = SortingOptions {
    SortableBy(\BookEntity.$title)
    SortableBy(\BookEntity.$dateRead)
    SortableBy(\BookEntity.$datePublished)
  }

  static var properties = EntityQueryProperties {
    Property(keyPath: \BookEntity.title) {
      EqualToComparator { NSPredicate(format: "title = %@", $0) }
      ContainsComparator { NSPredicate(format: "title CONTAINS %@", $0) }
    }
    Property(keyPath: \BookEntity.datePublished) {
      LessThanComparator { NSPredicate(format: "datePublished < %@", $0 as NSDate) }
      GreaterThanComparator { NSPredicate(format: "datePublished > %@", $0 as NSDate) }
    }
    Property(keyPath: \BookEntity.dateRead) {
      LessThanComparator { NSPredicate(format: "dateRead < %@", $0 as NSDate) }
      GreaterThanComparator { NSPredicate(format: "dateRead > %@", $0 as NSDate) }
    }
  }

  func entities(for identifiers: [UUID]) async throws -> [BookEntity] {
    identifiers.compactMap { identifier in
      Database.shared.book(for: identifier)
    }
  }

  func suggestedEntities() async throws -> [BookEntity] {
    Model.shared.library.books.map { BookEntity(id: $0.id, title: $0.title) }
  }

  func entities(matching string: String) async throws -> [BookEntity] {
    Database.shared.books.filter { book in
      book.title.lowercased().contains(string.lowercased())
    }
  }

  func entities(
    matching comparators: [NSPredicate],
    mode: ComparatorMode,
    sortedBy: [Sort<BookEntity>],
    limit: Int?
  ) async throws -> [BookEntity] {
    Database.shared.findBooks(matching: comparators, matchAll: mode == .and, sorts: sortedBy.map { (keyPath: $0.by, ascending: $0.order == .ascending) })
  }
}
```

## User interactions

- [Dialog][IntentDialog]
  - spoken or textual response for intent
  - `needsValueDialog` on `@Parameter` for example
  - `Value` (something) result

- [Snippet view][snippetviewtype]
  - visual equivalent of dialog
  - lets you add a visual representation (a SwiftUI view) to the result of your intent
  - return [`.finished(dialog:view:)`][finished(dialog:view:)] as the `IntentPerformResult`

```swift
struct AddBook: AppIntent {
  static var title: LocalizedStringResource = "Add Book"

  @Parameter(title: "Title")
  // ...

  func perform() async throws -> some PerformResult {
    // ..

    return .finished(value: book) {
      CoverView(book: book) // 👈🏻
    }
  }

  enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
    ...
  }
}
```

- Request Value - ask user for value via `requestValue(String)`
- Disambiguation - let user choose via [`requestDisambiguation(among:, dialog:)`][requestDisambiguation(among:dialog:)]
- Confirmation
  - request confirmation via [`requestConfirmation(for:dialog:)`][requestConfirmation(for:dialog:)] or `requestConfirmation(output: .result(value:dialog:))`
  - Last variant also supports showing a preview

```swift
struct AddBook: AppIntent {
  static var title: LocalizedStringResource = "Add Book"

  @Parameter(title: "Title")
  // ...

  func perform() async throws -> some PerformResult {
    let books = // ... fetch books by reading @Parameter values

    if books.count > 1 { // 👈🏻 too many matches! request disambiguation 👇🏻
      let chosenAuthor = try await $authorName.requestDisambiguation(among: books.map { $0.authorName }, dialog: "Which author?")
    }
    return .finished
  }

  enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
    ...
  }
}
```

## Architecture and lifecycle

- In-app
  - No need for a framework / duplicated code
  - No cross-process coordination
  - Higher memory limits
  - Ability to play audio
  - Run in foreground if you set [`openAppWhenRun`][openAppWhenRun]
  - Implement multi-scene support for best performance

- Extension target
  - Light-weight
  - Best performance
  - Focus filter intents, run immediately when Focus changes
  - Create by choosing <kbd>app intents extension</kbd> template in Xcode

- Your code is the only source of truth
- Xcode extracts App Intent at build-time
- Compile AppIntents code directly into app / extension (not through package)
- Upgrading to App Intents
  - keep using SiriKit intents for Widgets or Siri domains
  - Others should upgrade

[appintents]: https://developer.apple.com/documentation/appintents
[app-shortcuts]: https://developer.apple.com/documentation/appintents/app-shortcuts
[app-intents]: https://developer.apple.com/documentation/appintents/app-intents
[app-entities]: https://developer.apple.com/documentation/appintents/app-entities
[IntentResult]: https://developer.apple.com/documentation/appintents/intentresult
[IntentError]: https://developer.apple.com/documentation/appintents/intenterror
[appintent/perform()]: https://developer.apple.com/documentation/appintents/appintent/perform()
[AppEnum]: https://developer.apple.com/documentation/appintents/appenum
[IntentParameter]: https://developer.apple.com/documentation/appintents/intentparameter
[AppEntity]: https://developer.apple.com/documentation/appintents/appentity
[openentityintent]: https://developer.apple.com/documentation/appintents/openentityintent
[entity-queries]: https://developer.apple.com/documentation/appintents/entity-queries
[property-comparators]: https://developer.apple.com/documentation/appintents/property-comparators
[EntityPropertyQuery]: https://developer.apple.com/documentation/appintents/entitypropertyquery
[requestDisambiguation(among:dialog:)]: https://developer.apple.com/documentation/appintents/intentparameter/requestdisambiguation(among:dialog:)
[requestConfirmation(for:dialog:)]: https://developer.apple.com/documentation/appintents/intentparameter/requestconfirmation(for:dialog:)
[IntentDialog]: https://developer.apple.com/documentation/appintents/intentdialog
[finished(dialog:view:)]: https://developer.apple.com/documentation/appintents/intentperformresult/finished(dialog:view:)-5aejs
[snippetviewtype]: https://developer.apple.com/documentation/appintents/intentperformresult/snippetviewtype
[openAppWhenRun]: https://developer.apple.com/documentation/appintents/appintent/openappwhenrun-3u9y8
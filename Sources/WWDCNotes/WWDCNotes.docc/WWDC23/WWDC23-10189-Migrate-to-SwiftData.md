# Migrate to SwiftData

Discover how you can start using SwiftData in your apps. We’ll show you how to use Xcode to generate model classes from your existing Core Data object models, use SwiftData alongside your previous implementation, or even completely replace your existing solution.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10189", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(lordmooch)
   }
}



**Presenter:** Luvena Huo, SwiftData Engineer

**Super power:** Movement. Seamlessly from one state to the next.

**Length:** 11m, Supports Copy Code

# Migration to SwiftData

- How to migrate your app from Core Data to SwiftData
- SwiftData - Swift native persistence framework
- SwiftData - can co-exist with Core Data

# Common use cases

- Complete transition - replace Core Data with SwiftData
- Incremental transition - use Core Data and SwiftData side-by-side
- The first step is to generate SwiftData model classes
- Then decide; complete or incremental - depends on your use case

# Generate model classes

- SwiftData is a transition to using Swift code to generate your schema
- Can use Core Data managed object model to help generate SwiftData models
- In Xcode:
— Select object model file
— Select Editor menu
— Click on Create SwiftData Code…
- This approach is not required if you are using SwiftData from scratch
- Once created, you will have a Swift file for each entity you migrated
- These will use the new `@Model` macro
- They will use `@Relationship` for properties that relate to properties on other models

# Complete adoption

- You will be **replacing** your Core Data stack with your SwiftData stack
- Swift native - allows for more legible code for persisting data
- Implicitly manages some features

## Considerations

- Core Data model designs **must** be supported in SwiftData
- Must have an exact match of entity name and properties in SwiftData

## Set up persistent stack

- Generate model classes
- Delete the Core Data managed object model file
- Also, can delete the persistence file previously used to help setup the Core Data stack
- Everything now managed through the new SwiftData model classes

## Set up ModelContainer

- This is the `ModelContainer` for the SwiftData stack
- `.modelContainer` is a modifier
- It ensures all models in a group can access the same persistent container
- You add it to a `View`

```Swift
@main
struct TripsApp: App {
	var body: some Scene {
		WindowGroup {
		ContentView()
		}
		.modelContainer(
			for: [Trip.self, BucketListItem.self, LivingAccommodation.self])
	}
}
```

- This also sets a default `ModelContext` in the Environment
- `ModelContext` is used to track changes to instances of an app’s types
- Can be read from any `Scene` or `View` as follows:

```Swift
@Enviroment(\.modelContext) private var modelContext
```

## Object creation
- **Before in Core Data…**

```Swift
@Enviroment(\.managedObjectContext) private var viewContext

let newTrip = Trip(context: viewContext)
newTrip.name = name
newTrip.destination = destination
newTrip.startDate = startDate
newTrip.endDate = endDate
```

- **After in SwiftData…**

```Swift
@Environment(\.modelContext) private var modelContext

let trip = Trip(name: name, destination: destination, startDate: startDate, endDate: endDate)

modelContext.insert(object: trip)
```

## Saving changes
- SwiftData has an implicit save
  - Triggered on UI lifecycle events
  - Or on timer after context is changed - if possible
- Can remove Core Data explicit saves and rely on SwiftData saving when context changes

## Fetching data
- Replace Core Data’s `@FetchRequest` with SwiftData’s `@Query`
- For example - fetch full list of Trip models

```Swift
@Query(sort: \.startDate, order: .forward)

var trips: [Trip]
```

- Can also use predicates with `@Query` for finer control over the data returned

# Incremental adoption

- SwiftData will coexist with Core Data
- If a full conversion is not possible consider incremental adoption
- Two separate stacks talking to the **same persistent store**
- No need to rewrite existing Core Data code in order to start using new SwiftData code
- Ensure that both stacks are writing to the same URL
- Peristent History Tracking **must be turned on**
- SwiftData automatically turns this on but Core Data does not
- The store will be placed into read-only mode if Persistent History Tracking is not turned on
- Use incremental adoption when
  - Backwards compatibility required
  - Resource constraints may dictate only new work uses SwiftData

## Considerations

- Namespaces - names of SwiftData classes and Core Data classes must not collide!
- One name must change
  - Only the class name changes
  - The entity name it refers to will stay the same
- Keep schemas in sync in both SwiftData and Core Data
  - The schemas cannot diverge
  - Properties and relationships must be added in the exact same way in both schemas
  - This ensures entity hashes remain the same
  - If entity hashes differ it could trigger unwanted migrations and unintended consequences
- Schema versioning is important so SwiftData can evaluate the differences
- Can use SwiftData in UIKit and AppKit
  - Coexistence
    - bind UIKit code to Core Data
    - can work in parallel with SwiftData
  - Treat SwiftData classes as Swift classes and wrap your Swift code with UIKit code instead

# Takeaways

- Migration to SwiftData is flexible
- Complete migration - abandon Core Data, wish it well, but don’t turn back
- Incremental adoption - SwiftData and Core Data can co-exist, but watch out for jealousy over time
- Choose the best option for your use case

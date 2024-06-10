# Dive deeper into SwiftData

Learn how you can harness the power of SwiftData in your app. Find out how ModelContext and ModelContainer work together to persist your app’s data. We’ll show you how to track and make your changes manually and use SwiftData at scale with FetchDescriptor, SortDescriptor, and enumerate.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10196", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Nick Gillett, SwiftData Engineer

**NOTE: Watch "Meet SwiftData" and "Model your schema with SwiftData" first.**

## Configuring persistence

### Model

- Uses types you already use.
- `@Model` macro describes the schema and is used for instances in code.
- Inferred or explicit structure.
- Offers deep customization.

> The Schema is applied to a class called the `ModelContainer` to describe how data should be persisted.

> The `ModelContainer` consumes the Schema to generate a database that can hold instances of the `Model` classes.

> When working with instances of a `Model` class in code, those instances are linked to a `ModelContext` which tracks and manages their state in memory.

### ModelContainer

- The bridge between the Schema and its persistence.
- It holds descriptions about how objects are stored, like whether they're in memory, or on disk.
- Knows about versioning, migration, and graph separation.

```swift
// ModelContainer initialized with just Trip
let container = try ModelContainer(for: Trip.self)

// SwiftData infers related model classes as well
let container = try ModelContainer(
    for: [
        Trip.self, 
        BucketListItem.self, 
        LivingAccommodation.self
    ]
)
```

The `ModelContainer` is added to a `View` or `Scene` by using the `.modelContainer()` modifier.

For more advanced use cases the `ModelContainer` can be instantiated with a `ModelConfiguration`.

### ModelConfiguration

- Describes the persistence of a Schema.
- Controls where data is stored, like in memory for transient data or on disk for persistent data.
- Can use a specific file URL chosen by you, or it can generate one automatically using the entitlements of your application like the group container entitlement.
- Can describe that a persistence file should be loaded in a read only mode, preventing writes to sensitive or template data.
- Applications that use more than one CloudKit container can specify it as part of the `ModelConfiguration``for a Schema.

Here is an example, where some schemas (trip, bucket list item and living accommodation) are in one store and others (person, address) are in another store:
```swift
let fullSchema = Schema([
    Trip.self,
    BucketListItem.self,
    LivingAccommodations.self,
    Person.self,
    Address.self
])

let trips = ModelConfiguration(
    schema: Schema([
        Trip.self,
        BucketListItem.self,
        LivingAccommodations.self
    ]),
    url: URL(filePath: "/path/to/trip.store"),
    cloudKitContainerIdentifier: "com.example.trips"
)

let people = ModelConfiguration(
    schema: Schema([Person.self, Address.self]),
    url: URL(filePath: "/path/to/people.store"),
    cloudKitContainerIdentifier: "com.example.people"
) 

let container = try ModelContainer(for: fullSchema, trips, people)
```

> With the power of ModelConfiguration, it's easy to describe the persistence requirements of your application, no matter how complicated they may be.

## Track and persist changes

### ModelContext

- Tracks objects in use
- Propagates changes to `ModelContainer`
- Clear changes with rollback or reset
- Undo/redo support
- Autosave

> When we use the `.modelContainer()` modifier in view or scene code, it prepares the application's environment in a specific way. The modifier binds the new `\.modelContext` key in the environment to the container's mainContext.

> The main context is a special `MainActor`-aligned model context intended for working with ModelObjects in scenes and views. By using the model context from the environment, view code has easy access to the context used by the `@Query` here so that it can perform actions like delete here.

```swift
struct ContentView: View {
    @Query var trips: [Trip]   <------- Fetch objects
    @Environment(\.modelContext) var modelContext
  
    var body: some View {
        NavigationStack (path: $path) {
            List(selection: $selection) {
                ForEach(trips) { trip in
                    TripListItem(trip: trip)
                        .swipeActions(edge: .trailing) {
                            Button(role: .destructive) {
                                modelContext.delete(trip)   <------- Delete object
                            } label: {
                                Label("Delete", systemImage: "trash")
                            }
                        }
                }
                .onDelete(perform: deleteTrips(at:))
            }
        }
    }
}
```

#### Undo/redo

- Automatically registers actions
- `.modelContainer()` uses the environment's `\.undoManager`
- Support standard system gestures

When undo is enabled, "shake-to-undo" and three finger swipes can be used to undo or redo changes with no additional code.

```swift
@main
struct TripsApp: App {
   @Environment(\.undoManager) var undoManager
   var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, isUndoEnabled: true)
    }
}
```

#### Autosave

- Main context automatically saves
- Will save in response to system events like an application entering the foreground or background
- Will also periodically save as an application is used

Autosave is enabled by default, but can be disabled if desired using the `.modelContainer()` modifier's `isAutosaveEnabled` argument.

> Autosave is disabled for model contexts created by hand.

```swift
@main
struct TripsApp: App {
   var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, isAutosaveEnabled: false)
    }
}
```

## Modeling at scale

> Tasks like working with data on a background queue, syncing with a remote server or other persistence mechanism, and batch processing all work with model objects, frequently in sets or graphs.

### `FetchDescriptor`

`FetchDescriptor` uses the new `Predicate` macro. 

> `Predicate` uses the Models you create and SwiftData uses the Schema generated from those models to translate these predicates into database queries. `FetchDescriptor` combines the power of the new Foundation `Predicate` macro with the Schema to bring compiler validated queries to persistence.

> `FetchDescriptor` and related classes, like `SortDescriptor`, use generics to form the result type and tell the compiler about the properties of the model you can use. There are a number of tuning options you've come to know and love, like offset and limit, as well as parameters for faulting and prefetching.

Fetching objects is easy and requires no casting:
```swift
let context = self.newSwiftContext(from: Trip.self)
var trips = try context.fetch(FetchDescriptor<Trip>())
```

Fetching object with a predicate:
```swift
let context = self.newSwiftContext(from: Trip.self)
let hotelNames = ["First", "Second", "Third"]

var predicate = #Predicate<Trip> { trip in
    trip.livingAccommodations.filter {
        hotelNames.contains($0.placeName)
    }.count > 0
}

var descriptor = FetchDescriptor(predicate: predicate)
var trips = try context.fetch(descriptor)
```

### Enumerate function on `ModelContext`

> Designed to help make the foiblesome pattern of batch traversal and enumeration implicitly efficient by encapsulating the platform best practices at a single call site.

- Works great with `FetchDescriptor`s regardless of their complexity
- Implements platform best practices for traversals like batching and mutation guards
- Batch size is default set to 5000 objects
  - Could be set to 10000 to reduce I/O at the expense of memory growth
  - For heavier data graphs it could be lowered at the expense of more I/O

> One of the most frequent causes of performance issues with large traversals is mutations that are trapped in the context during the enumeration. `allowEscapingMutations` tells enumerate that this is intentional, when not set, enumerate will throw if it discovers that the ModelContext performing the enumeration is dirty, preventing it from freeing objects that were already traversed.

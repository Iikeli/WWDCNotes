# Meet SwiftData

SwiftData is a powerful and expressive persistence framework built for Swift. We’ll show you how you can model your data directly from Swift code, use SwiftData to work with your models, and integrate with SwiftUI. By [Miká Kruschel](<doc:mikakruschel>).

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/videos/play/wwdc2023/10187", purpose: link, label: "Watch Video")
   
   @Contributors {
      @GitHubUser(mikakruschel)
      @GitHubUser(multitudes)
   }
}

@Comment { TODO: Add chapter links during CI step before building docs. }

## Key Takeaways

📦 `@Model` macro for class \
🔑 `@Attribute(.unique)` for fields \
🔁 `@Relationship` for inverses/deletion \
❌ `@Transient` to exclude \
🌐 `@Environment(.modelContext)` context \
🔍 `#Predicate` & `FetchDescriptor` in code \
🔎 `@Query(sort:order:)` in view

@Row {
   @Column {
      @Image(source: "WWDC23-10187-SwiftData")
   }
   @Column {
      @Image(source: "WWDC23-10187-SwiftData")
   }
}
@Row {
   @Column {
      @Image(source: "WWDC23-10187-SwiftData")
   }
   @Column {
      @Image(source: "WWDC23-10187-SwiftData")
   }
}


## Chapters

[0:00 - Intro](https://developer.apple.com/wwdc23/10187)  
[1:07 - Using the model macro](https://developer.apple.com/wwdc23/10187?time=67)  
[3:17 - Working with your data](https://developer.apple.com/wwdc23/10187?time=197)  
[7:02 - Use SwiftData with SwiftUI](https://developer.apple.com/wwdc23/10187?time=422)  
[8:10 - Wrap-up](https://developer.apple.com/wwdc23/10187?time=490)

![SwiftData](WWDC23-10187-SwiftData)

# Intro

- Using the model macro  
- Working with your data  
- Use SwiftData with SwiftUl  

SwiftData is a powerful framework for data modeling and management. It uses Swift's new macro system to create a seamless API experience.  It is naturally integrated with SwiftUI and works with other platform features, like CloudKit and Widgets.

## Using the model macro 

### @Model

- Powerful new Swift macro  
- Define your schema with code  
- Add SwiftData functionality to model types  

@Model is a Swift macro that helps to define the model's schema from the Swift code. SwiftData schemas are normal Swift code, but when needed, can be used to annotate properties with additional metadata. Using this schema, SwiftData adds powerful functionality to the model objects. 

Add `import SwiftData` and `@Model` to the model class and the schema is generated.

```swift
// Adding @Model to Trip

import SwiftData

@Model
class Trip {
var name: String 
var destination: String 
var endDate: Date 
var startDate: Date

var bucketList: [BucketListItem]? = []
var livingAccommodation: LivingAccommodation?
}
```

Models in SwiftData are the source of truth and will transform the class' stored properties into persisted properties. 

### Attributes

SwiftData natively adapts value type properties to be used as attributes. These properties include basic value types, like string, int, and float. They can also include more complex value types, such as structs, enums, and codable types too, including collections.

### Relationships

Relationships are inferred from reference types  

- Other model types  
- Collections of model types  

SwiftData models reference types as relationships.

###  Additional Metadata

@Model will modify all the stored properties on a custom type using metadata like: 

- `@Attribute(.unique)` for uniqueness constraint  
- `@Relationship` for choice of inverses and delete propagation rules  
- `@Transient` to exclude property from model  

@Relationship can be used to control the choice of inverses and specify delete propagation rules. These change the behaviors of links between models. Use the Transient macro to tell SwiftData not to include specific properties. 
Here is an example.

```swift
// Providing additional metadata

import SwiftData

@Model
class Trip {
    @Attribute(.unique) var name: String 
    var destination: String 
    var endDate: Date 
    var startDate: Date
    
    @Relationship(.cascade) var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

To learn more about SwiftData modeling, check out the session:  
[Model your schema with SwiftData -                WWDC23](https://developer.apple.com/videos/play/wwdc2023/10195)  

# Working with your data

## Model container

The Model container provides the persistent backend  

- Customized with configurations. Use the default settings just by specifying the schema.  
- Provides schema migration options  
- Create a container by specifying the list of model types to store
- optionally provide `ModelConfiguration` with an url, CloudKit and group container identifiers, and migration options

```swift
let container = try ModelContainer(for: [Trip.self, LivingAccommodation.self], configurations: ModelConfiguration(url: URL("path")))
```
- Or in SwiftUI with the modifier

```swift
.modelContainer(for: [Trip.self, LivingAccommodation.self])`
```

## ModelContext

With the container set up, fetch and save data with model contexts. Also can be used with a SwiftUI's view and scene modifiers to set up a container and have it in the view's environment.

```swift
import SwiftData 
import SwiftUI

@main
struct TripsApp: App {
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        } 
        .modelContainer(for:
            [Trip.self, 
            LivingAccommodation.self])
        )
    }
}
```

Model contexts observe all the changes to the models and provide many of the actions to operate on them.

- Tracking updates  
- Fetching models  
- Saving changes  
- Undoing changes

In SwiftUI, generally the modelContext is from the view's environment.

```swift
import SwiftData 
import SwiftUI

struct ContextView : View {
    @Environment(\.modelContext) private var context
}

// or outside the view hierarchy, a shared main actor bound context,
let context = container.mainContext

// or instantiate new contexts for a given model container.
let context = ModelContext(container)
```

## Fetching Data

New in iOS 17, predicate works with native Swift types and uses Swift macros for strongly typed construction. It's a modern replacement for NSPredicate.   

New Swift native types:  
- Predicate  
- FetchDescriptor  
Improvements to SortDescriptor  

## Predicate

- Fully type checked.
- `#Predicate` construction instead of text parsing  
- Autocompleted keypaths.

Here are a few examples of building predicates.

```swift
// I can specify all the trips whose destination is New York.
let tripPredicate = #Predicate<Trip> {
    $0.destination == "New York"
}

// I can narrow our query down to just trips about birthdays
let tripPredicate = #Predicate<Trip> {
    $0.destination == "New York" && $0.name.contains("birthday")
}

// and I can specify we're only interested in trips planned for the future, as opposed to any of our past adventures. 

let today = Date()
let tripPredicate = #Predicate<Trip> { 
    $0.destination == "New York" &&
    $0.name.contains("birthday") &&
    $0.startDate > today
}
```

We can use the new FetchDescriptor type and instruct our ModelContext to fetch those trips.

```swift
let descriptor = FetchDescriptor<Trip>(predicate: tripPredicate)

let trips = try context.fetch (descriptor)
```

## SortDescriptor

- Updated to support all Comparable types  
- Swift native keypaths..

Swift SortDescriptor works together with FetchDescriptor and is getting some updates to support native Swift types and keypaths

```swift
let descriptor = FetchDescriptor<Trip>(
    sortBy: SortDescriptor(\Trip.name),
    predicate: tripPredicate
)
let trips = try context.fetch(descriptor)
```

## More FetchDescriptor options

- relationships to prefetch  
- result limits  
- exclude unsaved changes and more  

## Modifying Data

Basic operations

- Inserting  
- Deleting  
- Saving  
- Changing

Delete persistent objects marking them for deletion, and committing them to the persistent container.

```swift
var myTrip = Trip(name: "Birthday Trip", destination: "New York")

// Insert a new trip
context.insert(myTrip)

// Delete an existing trip
context.delete(myTrip)

// Manually save changes to the context
try context.save()
```

- The `@Model` macro modifies stored properties for change tracking and observation  
- Updated automatically by the ModelContext

To learn more about SwiftData containers and contexts and driving its operations, check out the session:  
[Dive deeper into SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10196)  

# Use SwiftData in SwiftUI

- Seamless integration with SwiftUI  
- Easy configuration  
- Automatically fetch data and update views  

## View modifiers

- Configure data store with `.modelContainer` which is propagated throughout SwiftUI environment  
- Fetching in SwiftUI with **`@Query`**  
- No need for `@Published` and SwiftUI automatically updates  


```swift
// @Query

import SwiftData
import SwiftUI

struct ContentView: View  {
    @Query(sort: \.startDate, order: .reverse) var trips: [Trip]
    @Environment(\.modelContext) var modelContext
    
    var body: some View {
       NavigationStack() {
          List {
             ForEach(trips) { trip in 
                 // ...
             }
          }
       }
    }
}
```

## Observing changes

- No need for @Published  
- SwiftUl automatically refreshes  

SwiftData supports the all-new observable feature for your modeled properties. SwiftUI will automatically refresh changes on any of the observed properties. SwiftUI and SwiftData work hand in hand  

Learn more about using these frameworks together in our session:  
[Build an app with SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10154)

![SwiftData 2](WWDC23-10187-SwiftData2)


@Comment { TODO: Add resources section to end during CI step before building docs. }

## Resources

[Adopting SwiftData for a Core Data app](https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app)  
[Have a question? Ask with tag wwdc2023-10187](https://developer.apple.com/forums/create/question?&tag1=719030&tag2=698030)  
[Search the forums for tag wwdc2023-10187](https://developer.apple.com/forums/tags/wwdc2023-10187)  
[SwiftData](https://developer.apple.com/documentation/SwiftData)

@Comment { TODO: Add related videos section to end during CI step before building docs. }


# Related Sessions

[Build an app with SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10154)  
[Discover Observation in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10149)  
[Dive deeper into SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10196)  
[Migrate to SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10189)  
[Model your schema with SwiftData -                WWDC23](https://developer.apple.com/videos/play/wwdc2023/10195)  
[What’s new in Swift - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10164)  
[What’s new in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10148)


@Comment { TODO: Add written by section to end during CI step before building docs. }

## Written by

@Row(numberOfColumns: 5) {
   @Column { ![Profile image of Miká Kruschel](https://avatars.githubusercontent.com/u/20423069?v=4) }
   @Column(size: 4) {
      ## [Miká Kruschel](<doc:mikakruschel>)
      
      Some basic information about Miká gathered from GitHub.
      
      [Contributed Notes](<doc:mikakruschel>)
      [GitHub](https://)
      [Website](https://)
      [Mastodon](https://)
   }
}

@Comment { TODO: Add legal notice section to end during CI step before building docs. }

## Related Sessions

@Links(visualStyle: list) {
   - <doc:WWDC20>
   - <doc:Contributing>
}


@Small {
   **Legal Notice**
   
   All content copyright © 2012 – 2024 Apple Inc. All rights reserved.
   Swift, the Swift logo, Swift Playgrounds, Xcode, Instruments, Cocoa Touch, Touch ID, FaceID, iPhone, iPad, Safari, Apple Vision, Apple Watch, App Store, iPadOS, watchOS, visionOS, tvOS, Mac, and macOS are trademarks of Apple Inc., registered in the U.S. and other countries.
   This website is not made by, affiliated with, nor endorsed by Apple.
}

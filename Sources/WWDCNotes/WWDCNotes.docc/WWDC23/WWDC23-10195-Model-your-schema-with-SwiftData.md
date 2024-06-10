# Model your schema with SwiftData

Learn how to use schema macros and migration plans with SwiftData to build more complex features for your app. We’ll show you how to fine-tune your persistence with @Attribute and @Relationship options. Learn how to exclude properties from your data model with @Transient and migrate from one version of your schema to the next with ease.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10195", purpose: link, label: "Watch Video (9 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## Utilizing schema macros

- `import SwiftData` and mark `class` with `@Model` to create schema
- Use `@Attribute(.unique)` to make field unique, if already exists, then "upsert" will happen, updating existing data (insert -> update)
- Uniqueness available for primitive value types: Numeric, String, UUID – can also decorate relationships
- Renaming variables without any attributes generates new properties and deletes old
- To keep data but rename field, specify `@Attribute(originalName: "oldName")` to map data
- `@Attribute` also supports storing large data externally, supplying support for `Transformable`s, integrate with Spotlight, modify hash
- SwiftData implicitly discoveres (inverse) relationships between models for fields like `var bucketList: [BucketListItem]? = []`
- The default deletion strategy for relationships when no annotations provided is "nil out"
- To delete relationships alongside, specify attribute `@Relationship(.cascade)` explicitly
- `@Relationship` modifier also supports `originalName` and constraints on `toMany` for min/max count constraints, plus modifying hash
- To exclude a stored property from the schema, annotate it with `@Transient` – a default value is required

## Evolving schemas

- Define an enum conforming to `VersionedSchema` and put your models inside the enum (used as a namespace)
- Order versions with `SchemaMigrationPlan`
- Define migration stage: Lightweight (no code required), Custom (code needed)
- Provide `migrationPlan` to `ModelContainer`

Example for `VersionedSchema`:
```Swift
enum SampleTripsSchemaV2: VersionedSchema {
  static var models: [any PersistentModel.Type] {
    [Trip.self, BucketListItem.self, LivingAccommodation.self]
  }
  
  @Model
  final class Trip {
    @Attribute(.unique)
    var name: String

    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
  }
  
  // other models
}
```

Example for `SchemaMigrationPlan`:
```Swift
enum SampleTripsMigrationPlan: SchemaMigrationPlan {
  static var schemas: [any VersionedSchema.Type] {
    [SampleTripsSchemaV1.self, SampleTripsSchemaV2.self, SampleTripsSchemaV3.self]
  }

  static var stages: [MigrationStage] { [migrateV1toV2] }

  static let migrateVltoV2 = MigrationStage.custom(
    fromVersion: SampleTripsSchemaV1.self, 
    toVersion: SampleTripsSchemaV2.self,
    willMigrate: { context in
      let trips = try? context.fetch(FetchDescriptor<SampleTripsSchemaV1.Trip> ())
      // De-duplicate Trip instances here..
      try? context.save ()
    },
    didMigrate: nil
  )
}
```

Example for passing migration plan to model container:
```Swift
struct TripsApp: App {
  let container = ModelContainer(for: Trip.self, migrationPlan: SampleTripsMigrationPlan.self)
    
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
    .modelContainer(container)
  }
}
```

# What's new in HealthKit

Bring the latest HealthKit features to your health & fitness app. We'll show you how to capture more detailed sleep data through sleep stages, track swim-bike-run and interval workouts with the enhanced Workout API, and save vision prescriptions — including an image of the physical prescription — directly to HealthKit while preserving privacy.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10005", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Sleep analysis

- Introducing sleep stages
- Apple Watch will automatically track all the different sleep stages you go through when you're asleep
- this data will be accessible from the Health app and saved in HealthKit
- apps can read/write sleep analysis data via [`HKCategoryTypeIdentifier.sleepAnalysis`][sleepanalysis]
- three sleep stages: REM, core, deep

```swift
public enum HKCategoryValueSleepAnalysis: Int {
  case inBed
  case asleep // ❌ removed
  case asleepUnspecified // 👈🏻 Replaces asleep, used when user is asleep but no sleep stage is specified
  case awake
  case asleepCore // 👈🏻 New
  case asleepDeep // 👈🏻 New
  case asleepREM  // 👈🏻 New
}
```

- new predicate [`HKCategoryValuePredicateProviding`][HKCategoryValuePredicateProviding] to access HealthKit samples, for example:

```swift
let stagePredicate = HKCategoryValueSleepAnalysis.predicateForSamples(.equalTo, value: .asleepREM)
let queryPredicate = HKSamplePredicate.sample(type: HKCategoryType(.sleepAnalysis), predicate: stagePredicate)
let sleepQuery = HKSampleQueryDescriptor(predicates: [queryPredicate], sortDescriptors: [])

// Run the query
let sleepSamples = try async sleepQuery.result(for: healthStore) // 👈🏻 available since iOS 15.4
```

## Swift async

- available since iOS 15.4
- see types that conform to the new [`HKAsyncQuery`][HKAsyncQuery] to see which queries support swift concurrency

## Workouts

- new workouts API in iOS 16 and watchOS 9 to make it possible to capture mixed activities workouts like [triathlon][Triathlon]
- each activity is represented via one [`HKWorkoutActivity`][HKWorkoutActivity]
- each `HKWorkoutActivity` comes with
  - its own [`HKWorkoutConfiguration`][HKWorkoutConfiguration], which includes its activity type
  - an array of [`HKWorkoutEvent`][HKWorkoutEvent]

- you can read statistics for each `HKWorkoutActivity` separately
- workout activities are not required to be contiguous
- between activities you can create a `HKWorkoutActivity` with type [`HKWorkoutActivityType.transition`][HKWorkoutActivityType.transition]
- all activities will be put together under the same [`HKWorkout`][HKWorkout] object
- new predicates for querying only a specific type of activity within workouts, e.g., [`predicateForWorkoutActivities(operatorType:quantityType:averageQuantity:)`][predicateForWorkoutActivities(operatorType:quantityType:averageQuantity:)]

### Heart rate recovery

- new Cardio Recovery data type [`HKQuantityTypeIdentifier.heartRateRecoveryOneMinute`][heartRateRecoveryOneMinute]
- additional context can be added as metadata

## Vision prescriptions

- from iOS 16, we can save glasses and contacts prescriptions in HealthKit
- [`HKObjectType.visionPrescriptionType()`][visionPrescriptionType()]
  - optionally we can store digital copy of the physical prescription
- new [`HKVisionPrescription`][HKVisionPrescription] class, along with [`HKGlassesLensSpecification`][hkglasseslensspecification] and [`HKGlassesPrescription`][HKGlassesPrescription] specification objects

[sleepanalysis]: https://developer.apple.com/documentation/healthkit/hkcategorytypeidentifier/1615425-sleepanalysis
[HKCategoryValuePredicateProviding]: https://developer.apple.com/documentation/healthkit/hkcategoryvaluepredicateproviding
[HKAsyncQuery]: https://developer.apple.com/documentation/healthkit/hkasyncquery
[Triathlon]: https://en.wikipedia.org/wiki/Triathlon
[HKWorkoutActivity]: https://developer.apple.com/documentation/healthkit/hkworkoutactivity
[HKWorkoutConfiguration]: https://developer.apple.com/documentation/healthkit/hkworkoutconfiguration
[HKWorkoutEvent]: https://developer.apple.com/documentation/healthkit/hkworkoutevent
[HKWorkoutActivityType.transition]: https://developer.apple.com/documentation/healthkit/hkworkoutactivitytype/transition
[HKWorkout]: https://developer.apple.com/documentation/healthkit/hkworkout
[predicateForWorkoutActivities(operatorType:quantityType:averageQuantity:)]: https://developer.apple.com/documentation/healthkit/hkquery/3929715-predicateforworkoutactivities
[heartRateRecoveryOneMinute]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3929728-heartraterecoveryoneminute
[visionPrescriptionType()]: https://developer.apple.com/documentation/healthkit/hkobjecttype/3916014-visionprescriptiontype
[HKVisionPrescription]: https://developer.apple.com/documentation/healthkit/hkvisionprescription
[hkglasseslensspecification]: https://developer.apple.com/documentation/healthkit/hkglasseslensspecification
[HKGlassesPrescription]: https://developer.apple.com/documentation/healthkit/hkglassesprescription
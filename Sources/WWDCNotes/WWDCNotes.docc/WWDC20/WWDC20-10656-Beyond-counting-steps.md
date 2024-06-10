# Beyond counting steps

Move beyond step counting in your app and give people a much richer understanding of their mobility. We’ll detail how you can take advantage of mobility metrics in iOS and watchOS to measure movement in more distinct and actionable ways. Learn about the latest HealthKit APIs for accessing mobility data, strategies for meaningful data aggregation, and how to interpret results for people using your app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10656", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Apple is introducing a category of metrics to HealthKit that captures the complex and important elements of human movement, all via iPhone (8 and later) and Apple Watch (4 and later).

## New Mobility metrics on iPhone

- Walking speed ([`.walkingspeed`][wsDoc]): Rate of travel over flat ground
- Step length ([`.walkingStepLength`][wslDoc]): Distance between feet when walking
- Double support time ([`.walkingDoubleSupportPercentage`][wdsDoc]): Percentage of time with two feet on the ground
- Walking asymmetry ([`.walkingAsymmetryPercentage`][wapDoc]): Percentage of walking when left and tight step time are different

## New Mobility metrics on Apple Watch

- Stair speed up ([`.stairAscentSpeed`][sasDoc]): Rate of vertical ascent on stairs
- Stair speed down ([`.stairDescentSpeed`][sdsDoc]): Rate of vertical descent on stairs
- Six-minute walk ([`.sixMinuteWalkTestDistance`][sixDoc]): Weekly estimate of distance that could be walked in six minutes on flat ground

## How to read HealthKit data

Request authorization:

```swift
/// Request authorization from the user 
self.healthStore = HKHealthStore() 

let typesToRead: Set = [ 
  HKObjectType.quantityType(forIdentifier:-walkingSpeed)!
]

self.healthStore.requestAuthorization(toShare: nil, read: typesToRead) { success, error in 
	if !success, let error = error { 
    print("Authorization failed: \(error.localizedDescription)") 
  }
}
```

Fetch the data of interest in the relevant time range:

```swift
// Select a time range 
let start = Calendar. current.date(byAdding: .day, value: -30, to: dateOfInjury)
let end = Calendar.current.date(byAdding: .day, value: 60, to: dateOfInjury)

let datePredicate = HKQuery.predicateForSamples(withStart: start, end: end, options: []) 

// Query walking speeds 
let walkSpeedType = HKSampleType.quantityType(forIdentifier: .walkingSpeed)!
let sortByStartDate = NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: true) 

let query = HKSampleQuery(
	sampleType: walkSpeedType, 
	predicate: datePredicate, 
	limit: HKObjectQueryNoLimit, 
  sortDescriptors: [sortByStartDate]) { _, samples, _ in
  visualizeWalkSpeeds(samples)
}
self.healthStore.execute(query) 
```

[wsDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3131040-walkingspeed
[wslDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3552088-walkingsteplength
[wdsDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3552087-walkingdoublesupportpercentage
[wapDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3552086-walkingasymmetrypercentage
[sasDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3552084-stairascentspeed
[sdsDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3552085-stairdescentspeed
[sixDoc]: https://developer.apple.com/documentation/healthkit/hkquantitytypeidentifier/3552083-sixminutewalktestdistance
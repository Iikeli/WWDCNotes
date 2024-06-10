# Getting started with HealthKit

HealthKit helps you build world-class health and fitness apps by centralizing health data from third-party apps, iPhone, Apple Watch, and external health devices. Discover how you can manage authorization and privacy around Health data, read and write data to the shared Health Store, and use HealthKit’s built-in queries to fetch data and calculate statistics for that data.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10664", purpose: link, label: "Watch Video (32 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What's HealtKit?

HealthKit is a framework that creates a central repository of all the user's health data, allowing applications to read data from it and also contribute data to it.

We can interact with health data on multiple devices including our iPhone, Apple Watch, and iCloud.

## How to build a HealthKit app

Three steps:

1. Setup HealthKit
2. Save data to HealthKit
3. Read and display HealthKit data 

### Setup HealthKit

- Enable HealthKit capability 
- Check whether the current platform (where the app is running on) supports HealthKit (via [`HKHealthStore.isHealthDataAvailable()`][isHealthDataAvailable])
- Create an [HKHealthStore][HKHealthStore], which is the access point for all data  managed by HealthKit

### Save data to HealthKit

All different types of health data are stored as [Health Samples][sample] ([`HKSample`][HKSample]).

Health Samples are all structured in a very similar pattern:

- they have an associated type of data that they represent
- the value of that type of data
- the time these health events occurred
- meta-data representing any additional information about these samples

To save data to HealthKit:

- Request write access to HealthKit
- Create Health samples
- Save the samples via the `HealthStore`

### Read data from HealthKit

HealthKit offers [queries][queries] to read HealthKit data

To read data from HealthKit:

- Create a query object, for each object you'll need to define:
  - the type of data that you want to read
  - a filter (predicate) that will restrict the results returned from your query

- Execute the query via the `HealthStore`

[queries]: https://developer.apple.com/documentation/healthkit/queries
[HKSample]: https://developer.apple.com/documentation/healthkit/hksample
[sample]: https://developer.apple.com/documentation/healthkit/samples
[HKHealthStore]: https://developer.apple.com/documentation/healthkit/hkhealthstore
[isHealthDataAvailable]: https://developer.apple.com/documentation/healthkit/hkhealthstore/1614180-ishealthdataavailable
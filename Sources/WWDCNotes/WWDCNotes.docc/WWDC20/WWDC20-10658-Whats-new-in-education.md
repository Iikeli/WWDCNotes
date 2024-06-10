# What's new in education

For over 40 years, Apple has been working with educators to create technologies for students, teachers, and school administrators and help them share in a rich and meaningful learning experience. Explore the breadth of Apple's education technologies, including classroom management apps and tools and developer frameworks for assessment and curriculum integration — and discover how your app can have a critical role in them all.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10658", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What's new in Schoolwork

[Schoolwork][Schoolwork] makes it easy for teachers to share materials, assign activities and track student progress.

- new design
- handout library
- handout detail view
- support for new activity metadata

If you want your app to integrate with Schoolwork, use the [`ClassKit`][classkit] framework

## What's new in ClassKit

ClassKit Catalog: 

- a central store of your app's public Activities
- using a web API, you can publish your Activities and they will be available from the Activity Chooser
- includes new metadata fields

## What's new in Apple School Manager

Apple School Manager (ASM) is Apple's web-based portal for schools to manage accounts, devices, and content, all from one place.

- classes created in ASM are automatically available in [Classroom][Classroom]
- any updates in ASM are synced down to both Classroom and Schoolwork
- allow managed Apple ID

## What's new in Classroom

- AirPlay class invitation
- Pinch to zoom student screens

## What's new in Shared iPad

- Shared iPad Temporary Session: 
 - enables schools to deploy devices with standard configurations that students can use without sign-in credentials.
 - all data is removed on sign out

## Automatic Assessment Configuration

[Automatic Assessment Configuration][ASC] (ASC) is a new framework used to tests. 

Thanks to ASC schools can now leverage the power of the Mac, while at the same time preventing students from using features that could give them an unfair advantage.

Features:

- Unified macOS and iOS API
- Catalyst support
- Restrict specific system features 

You need to apply for a new entitlement in order to use this framework.

[ASC]: https://developer.apple.com/documentation/automaticassessmentconfiguration
[Classroom]: https://apps.apple.com/us/app/classroom/id1085319084
[Schoolwork]: https://apps.apple.com/us/app/schoolwork/id1355112526
[classkit]: https://developer.apple.com/classkit/
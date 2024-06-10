# What’s New in ClassKit

The ClassKit framework helps you surface your app's valuable educational content for inclusion in a teacher's classroom curriculum. Get an overview of the ClassKit integration workflow, debugging instructor and student roles with the Schoolwork app, and new features designed to make publishing to ClassKit easier than ever.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/247", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What’s ClassKit?

It's a core part of Apple's education ecosystem.
The whole purpose of ClassKit is to allow your app to share a student's progress with a teacher.
ClassKit ensures that the data remains secure and only accessible to specific users like the teachers who have been granted privileges to access that data.

## How it works?

How does a teacher or the student see the progress in a ClassKit enabled app? 
Apple provides an application called [Schoolwork](https://apps.apple.com/us/app/schoolwork/id1355112526) for iPadOS.

Teachers use Schoolwork to create assignments called handouts and students receive these handouts from within Schoolwork.

When your application in response to a handout activity records progress, it is within Schoolwork that the teacher and student both can track this progress.

## How it really works? `CLSContexts`

The app can surface the activities that it supports via CLSContexts. 

e.g. For an app that teaches coding, each lesson might be an individual activity, and each lesson would have a corresponding CLSContext.

`CLSContext` help with tracking progress, quizzes, and time spent in each lesson.

`CLSContext`s are organized in a tree structure with child-parent relationships.

The first `CLSContext` is created by ClassKit, and from that one our app can create as many as pleased.

Each `CLSContext` should be a lesson-size topic, not entire section of the subject section. Same for quizzes.
This is because each Context can be assigned by the teacher to its students, therefore they cannot be too big.

ClassKit APIs often make reference to a Context Identifier Path:  
an identifier path is an array of strings, an array of identifiers of `CLSContexts` (the identifier is one of this object properties) from the root of the app context tree to the specific `CLSContext`.

When a teacher starts a new assignment he will be presented with each app `CLSContext` hierarchy. 

Once setup, the assignment will be sent to the students, that will be able to open our app directly from Schoolwork by tapping on the assignment.

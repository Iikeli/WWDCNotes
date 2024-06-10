# Build custom workouts with WorkoutKit

WorkoutKit makes it easy to create, preview, and schedule planned workouts for the Workout app on Apple Watch. Learn how to build custom intervals, create alerts, and use the built-in preview UI to send your own workout routines to Apple Watch.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10016", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(rogerluan)
   }
}



## Introduction to WorkoutKit

It allows you to create and customize all of the different workout types that a user can create in the Workout app within your own apps. It is also the bridge to help you bring these workouts into the Workout app for users to perform.

## Building a custom workout

Workouts are composed by 3 stages:

![Workout Stages][workout-stages]

- Each blue box in the imagre above is a step.
- Every step must have a goal. This can be a time or distance goal, if applicable, or also an open goal.
- Steps may have 0 or 1 alerts. Current alert types can be pace (average or current), cadence, power, and heart rate.
- Blocks are a group of steps, composing the 2nd stage in the image above.
- Blocks may contain any number of work and recovery steps, in any order. Blocks can also be repeatable.

Here's a code sample of a custom workout:

![Workout Code Sample][workout-code-sample]

- The `CustomWorkoutComposition` is called using `try` because it is validated upon creation.
- There's also a `WorkoutComposition` wrapper to perform additional operations such as APIs to import/export workout composition as files, which can be shared across devices.
- Export files using binary format because they're a lot smaller.
- There's an API to preview a workout file before importing it.
    - When triggered on watchOS, it will launch the Workout app and offer the user to start the workout, or import it.
    - On iOS, it will present a screen on top of your app, in a separate process:

![Workout Preview][workout-preview]

## Scheduled Workouts

- You can schedule workouts to be performed at a specific time, and plan your week ahead, for instance.
- Your app can schedule workouts and they will appear in a dedicated space at the top of the Workout app:

![Scheduled Workouts][scheduled-workouts]

- Scheduling workouts requires the user's permission, and syncing is handled locally.
- The user is able to see workouts scheduled in the next seven days and the previous seven days.
- You can query for scheduled workouts that the user has completed from your app. It contains scheduled date, and whether it was completed or not. They don't contain health data. If you want to pull the health data, refer to HealthKit APIs.

[workout-stages]: workout-stages.png
[workout-code-sample]: workout-code-sample.png
[workout-preview]: workout-preview.png
[scheduled-workouts]: scheduled-workouts.png

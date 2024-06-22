# Background execution demystified

Are you mystified about why and when your app gets background processing time? No need for a crystal ball! We’ll show you how to tackle the seven major factors that impact background runtime, and how you can integrate background modes that help your app come back to the foreground faster, run more smoothly, and reduce battery drain. 

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10063", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(mike011)
   }
}



This talk sets out to explain why when your app is in the background your `BGProgressingTask`, `BPAppRefreshTask`, background push, or `URLSession` isn't running as much as you might expect it to. If you are unfamiliar with these, checkout [`Advances in App Background Execution`](../../wwdc19/707/) from WWDC19.

## Factors Affecting Runtime

When your app is running it needs to achieve balance between what the system wants and want the customer wants. In the following diagram on the left side is what the system wants and on the right side is what the customer wants.
![][compare]

## The Top 7 Factors

There are many factors that affect the runtime of your app. Here are some examples

![][factors]

The following image lists the top 7 factors you should keep in mind that too keep your app up to date.

![][memory]

## Examples
Now the talk goes through different types of background activities and explains what they do and how they are constrained by the 7 factors.

## Background App Refresh
This is a way to ask the system to periodically run your app in the background to keep it up to date. This is the factor the customer has the most control over. They can decide if your app can run in the background by toggling the background app refresh switch in settings.

![][toggle]

The refreshes are not evenly spread out through out the day, because the device uses machine learning to figure out when you are most likely to use the app, and then it refreshes right before it expects you to use it.

Each app has an energy and data budget which is slowly depleted during the day.

The 7 factors can spread out the refresh rate even more. Here is an example of what the refresh rate could look like.

![][refresh]

## System Budgets

As a developer this is where you have the most control. There are various things you can do to minimize power consumption. Avoid bringing up unneeded hardware (gps, accelerometer). Do your work as quickly as possible by trying to serialize it as much as possible. Signal completion which allows the system to immediately suspend your app letting it enter a lower power state. Most of the time this involves calling a completion handler. You should also minimize the cellular data used. You can do this by downloading only what is critical. Try to keep your refresh under 100 kilobytes every time your app refreshes.

## Background Pushes

This is a silent alert to the application from the server. Only 6 factors come into play for how often these are done, app usage is excluded. Common question is how does rating limiting apply to background pushes? The system delays the delivery of some pushes. Everything is still delivered, just delayed.

## Background URL Session

They are simple downloads that are done in the background and are managed by the system. You can prevent these from using cellular data and you set it so your app will launch when the download has completed. There are 2 main types, discretionary and nondiscretionary. Discretionary background transfers are transfers you are OK with differing, leaving it up to the system to decide when the best time to transfer the files are. Nondiscretionary are the opposite, you want it now! Only 5 factors come into play for these, app usage and rate limiting are excluded. For discretionary transfers only app switcher and system budgets come into play.

## Background Processing Tasks

Gives your app several minutes of runtime at system friendly times to do maintenance work on your app. These are ran even if you don't bring your app to the foreground. 4 of the factors are applied to this mode, they are app usage, app switcher, background app refresh switch, and rate limiting.

## Summary

- Consider how top factors affect your app.
- Choose the right mode or modes for the job
- Reduce energy and data usage.

[compare]: WWDC20-10063-bed2
[memory]: WWDC20-10063-bed
[toggle]: WWDC20-10063-bed3
[refresh]: WWDC20-10063-bed4
[factors]: WWDC20-10063-bed5

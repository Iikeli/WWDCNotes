# Design for intelligence: Discover new opportunities

Learn how extensibility is key to surfacing the most important features of your app into new entry points of the operating system. And discover how — by breaking out of the constraints of a monolithic container — your app can see increased engagement through suggestions on the lock screen, in Calendar, and by enabling voice interactions.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10088", purpose: link, label: "Watch Video (5 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Intelligence Goals

The goal of intelligence is to:

- make your Apple products feel like they know you, they understand you.
- help you achieve more by accelerating you to a goal that you already have in mind
- help you discover more by enriching your life with meaningful content, people, places, and apps delivered to you at just the right time.
- save you clicks/taps.

## Integrate with the System

Your app can participate in intelligence by integrating with the system, which allows the system to see your app as much more than a monolithic container.

With this, your app gets many more entry points beyond the Home screen when we make suggestions on your behalf.

## Examples of integrations available to your app

- Shortcuts
- Widgets
- Sharing
- Siri Event Suggestions (your app events will automatically appear in Calendar, in maps, on the Lock screen, Siri will notify you when it's time to leave to go to your event, and more).

## Why Integrating with the System Intelligence

- Participating in intelligence is a great way to improve the traffic, utility, and visibility of your app:  
when a user engages on their first sharing suggestion for your app, they will on average share twice as much as they did before through your app.

- Some airlines saw 82 percent of notification check-ins come from their Siri event suggestion check-in action.
![][entryPointsImage]

- Some third-party apps are seen on average five times every day on the Lock screen and sharing in search and other entry points throughout the system.

[entryPointsImage]: entryPoints.png
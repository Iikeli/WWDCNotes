# Introduction to Siri Shortcuts

Siri Shortcuts are a powerful new feature in iOS 12 that allow your app to expose its functionality to Siri. This enables Siri to suggest your shortcut at relevant times based on various context. Shortcuts can also be added to Siri to run with a voice phrase on iOS, HomePod and watchOS. Learn how to expose shortcuts in your app using NSUserActivity and discover the benefits of creating custom intents with SiriKit for a richer user experience.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/211", purpose: link, label: "Watch Video (48 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- [Ari Weinstein][ariTweet], the first speaker in this session, is the creator of [Workflow][workflowApp], the app that was bought by Apple and now has become Shortcuts. Ari was also a popular jailbreaker back in the days (see Wall Street Journal interview [here][ariWS])
- Implementing `NSUserActivity` (iOS 8) allows us to represent content and actions to iOS. iOS represents these intelligently to the user in the home screen.
- iOS 12 supports custom siri extensions
- Siri Extensions can run and execute in the background.
- `NSUserActivity` is exposed in the siri settings and thus letting the user associate a spoken phrase with the activity (content or action).
If content can be deleted we must manage deleting NSUserActivity ‘donations’ to iOS. 
- Shortcuts suggestions use patterns to identify what suggestion to offer to the user, these suggestions take into account time of day, location, other signals provided by the developer (input parameters to the Intentions types)
- Three main steps for your app to create shortcuts:
  - Define the shortcut;
  - Donate the shortcut (a.k.a. let iOS know about your shortcut);
  - Handle shortcut (from the app or app extension)
- Every shortcut should: let the user do something faster (than just opening your app and do it manually), be of the user interest (the user should want to use this shortcut multiple times), be executable at any time;
- Shortcuts supports two APIs:
  - `NSUserActivity` (which represents the app state, can also be used for spotlight and handoff)
  - Siri Intents (tell the system what kind of action your app can perform);
- Use `NSUserActivity` when the shortcut will open your app or when the shortcut uses something you already index in spotlight or offer for handoff. Note that this can work only if that user activity is also eligible for search, otherwise it won’t work.
- Use Intents when it can run in the pipeline without launching your app, this can be fired via siri or a button.
- SiriKit offers a few default intents, but you can now also define your own, for each intent you must:
  - Choose a category (among the defaults one); 
  - Declare a title and a description (this will be displayed in the shortcut app);
  - Declare what parameter(s) it takes (like a list of items and a location);
  - Declare if it can run on the background or not (aka it can run the action with your app not open or not)
  - Declare your shortcut type(s) (up to 16)
  - Create a Siri extension for each intent (if you run it in the background)

[ariTweet]: https://twitter.com/AriX
[workflowApp]: https://my.workflow.is
[ariWS]: https://www.wsj.com/articles/SB124692204445002607
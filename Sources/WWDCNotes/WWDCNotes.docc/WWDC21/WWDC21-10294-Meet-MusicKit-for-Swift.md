# Meet MusicKit for Swift

MusicKit makes it easy to integrate Apple Music into your app. Explore the Swift-based framework: We’ll take you through the basic process of using MusicKit — including how to find, request, and play content — and show you how you can incorporate music subscription workflows into your app if someone hasn’t yet signed up to Apple Music.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10294", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



- MusicKit allows access to Apple Music API to search for content
- It also allows for playback in third party apps (user needs to be subscribed)
- Music items: `Album` with attributes, relationships and associations
- Use `album.with([.artists, .tracks, .relationAlbums])` for fetching relationships
- All calls are using Swift concurrency, so `try await` needed on call site
- Playback integrates with system, so media controls in lock screen work for example
- Requests done via `MusicCatalogResourceRequest<Album>(matching:equalTo:)`
- Then just call `try await request.response()` on the request object
- Contributor comment: *The networking structure here is interesting, maybe a new platform best practice!?*
- User consent is needed to access music listening history & music library (separately)
- Access is given once per device & per app, dialog has “yes” / “no” options
- As always, a usage description must be provided in `Info.plist` file
- Use `MusicAuthorization.request()` (async method) to get access in `MusicKit`
- API token for Apple Music API is automated by Xcode, just check the box in Connect
- 3 information to check regarding subscription capabilities:
    - can play catalog content, has cloud library enabled & can become subscriber

- `MusicSubscription.subscriptionUpdates` informs app when user subscribes
- Two players available, `SystemMusicPlayer` integrates with system more, `ApplicationMusicPlayer` allows for more customization for the app like playback queue
- Apple Music free trial can be started right within third party apps via system sheet
- New Affiliate program to get bonus if users subscribe via 3rd party app
- Class supports album customization, named `MusicSubscriptionOffer` (`.Options`)
- Possible use cases: background music in games, music during workouts, social media

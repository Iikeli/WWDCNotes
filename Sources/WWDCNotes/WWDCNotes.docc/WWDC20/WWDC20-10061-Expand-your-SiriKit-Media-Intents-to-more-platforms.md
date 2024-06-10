# Expand your SiriKit Media Intents to more platforms

Discover how you can enable Siri summoning for your music or audio app using SiriKit Media Intents. We’ll walk you through how to add Siri support to your music, podcast, or other audio service on more of our platforms, including HomePod and Apple TV, so people can start listening by just asking Siri. And learn about new APIs that let you support alternative results, helping people listen more quickly without leaving the Siri interface.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10061", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



SiriKit Media Intents allow you to add a wide variety of natural language queries to your application e.g.: 

- `Play music on AppName`
- `Play BandName on AppName`
- `Play song SongName on AppName`
- etc

## New platforms

From this year, SiriKit Media Intents come to

- HomePod, for more check [developer.apple.com/siri](https://developer.apple.com/siri)
- Apple TV

## New features

### Compact UI

Like the rest of the system, now SiriKit Media Intents will be handled in the system compact UI when intents are handled outside the app.

### Alternatives Support

After handling a request, in the compact UI there could be a `Maybe You Wanted` button that shows alternative songs to play instead of the current one.

This is achieved by calling 

```swift
INPlayMediaMediaItemResolutionResult.successes(with: mediaItems)
```

instead of 

```swift
INPlayMediaMediaItemResolutionResult.success(with: mediaItems[0])
```

when receiving a [`resolveMediaItems(for:with:)`][resolveMediaItems(for:with:)] call, e.g.:

```swifr
func resolveMediaItems(
  for intent: INPlayMediaIntent, 
  with completion: @escaping ([INPlayMediaMediaItemResolutionResult]) -> Void
) {
  let mediaSearch = intent.mediaSearch
  resolveMediaItems(for: mediaSearch) { optionalMediaItems in
    guard let mediaItems = optionalMediaItems else {
      return
    }
    completion(INPlayMediaMediaItemResolutionResult.successes(with: mediaItems))
  }
}
```

If the user taps on an alternative song, this will come to the app as a normal `INPlayMediaIntent`.

## Performance improvements

### In-app intent handling

- moves the intent handling in the app
- no need to launch your intents extension and your app to start a background audio session (just the app will suffice now)
- note that because it's launching the app this might result in a slower Siri response

### App prewarming

Currently the system will launch your app in the background for it to begin playback **after** handling a intent, with app prewarming the system will launch the app in the background along with the intent extension (therefore the app playback is ready to go when the intent is handled).

## Demo: ControlAudio

ControlAudio is a demo app introduced last year that the team has updated to support this year's new feature, download it [here](https://developer.apple.com/documentation/sirikit/media/managing_audio_with_sirikit).

[resolveMediaItems(for:with:)]: https://developer.apple.com/documentation/sirikit/inplaymediaintenthandling/3074275-resolvemediaitems
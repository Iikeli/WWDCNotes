# Introducing SiriKit Media Intents

iOS 13 enhances SiriKit by bringing all new support for audio content playback. See how to provide an excellent, hands-free experience for playing your music, audiobooks, podcasts, radio, and more. Dive into best practices for handling search terms, discover how to provide a complete experience with playback speeds, adding to playlists, and allowing customers to tell you if they like or dislike content.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/207", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}




## Possible Phrases For 3rd Party Apps

- `Tell <MyApp> that I love pop music`
- `Play Khalid on <MyApp>` 
- `I don't like this song` 
- `Add this to my library`


## Intents

- `INPlayMediaIntent` to allow playing audio
- `INAddMediaIntent` to add media items to playlists and libraries
- `INUpdateMediaAffinitiyIntent` to express affinity to media items
- `INSearchForMediaIntent` to search for specific media in your app
- Support for playback controls let users say `Play Billie Eilish shuffled in <MyApp>`
- Supported audio types: `Music`, `Podcasts`, `Audiobooks`, `Radio`
- Even search for unsupported media types possible 
- caveat: search queries will be untyped


## Handling SiriKit Media Requests

- Handling requests through Siri app extension
- Don't forget to add the Siri capability to your app
- Request processing involves `Resolve` > `Confirm` > `Handle`
- See [10:15](https://developer.apple.com/videos/play/wwdc2019/207/?time=610) or an extensive example about `Adding intents to your app`, `Specify supported intents and media types`, `Implement resolve, handle for INPlayMediaIntent and INAddMediaIntent`


## Best Practices

- If Siri support already added:
  - The new API uses existing code for handling background app launch
  - You need to add resolve methods
  - You need to update intents extension with supported media types

- Apple Watch
  - Foreground app launch via `INPLayMediaIntentResponseCode.continueInApp`
  - Intent is handled by your `WKExtensionDelegate`
  - Prefer on-device cache in your resolve method - only use network if absoloutey necessary

- Process results in resolve method `case insesnitive` becaue Siri might give you upper case results
- `Write an effective search method - be flexible` since 
  - Siri might understand certain media types, e.g. `video` if `video` is part of what the user searches
  - Siri might understand `sun` or `son` 

- Always populate `title`, `artist` and `type`in the returned `INMediaItem` since they all influence Siri's output 
- Handle error cases gracefully
  - Most common error is not found: `INPlayMediaMediaItemResolutionResult(InMediaItemResolutionResult.unsupported())`
  - List of possible errors in `INPlayMediaMediaItemUnsupportedReason`

- When you support playback controls in your app also support them in Siri
  - `Play <Song> on repeat in <MyApp>`
  - `Play <Playlist> on shuffle in <MyApp>`
  - `Resume <Podcast> at double speed in <MyApp>`
  - `Play <Artist> in <MyApp> next|later`

- User says `Play <MyApp>`
  - Don't ask what to play!
  - Choose something interesting automatically or resume the queue

- User vocabulary helps Siri recognize important named entities
- Global vocabulary is appropriate for global app terms

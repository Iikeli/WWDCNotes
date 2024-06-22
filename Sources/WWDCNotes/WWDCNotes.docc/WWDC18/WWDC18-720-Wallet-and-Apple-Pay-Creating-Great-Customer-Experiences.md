# Wallet and Apple Pay: Creating Great Customer Experiences

Get the latest news and updates from the Wallet and Apple Pay team. Learn how iPhone and Apple Watch can power innovative commerce experiences. Hear about the latest design best practices for Apple Pay. And discover how to create your own contactless passes for rewards cards, gift cards, tickets and more.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/720", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Why use passes?

- Easy to use contactlessly or with a barcode
- Synced across all devices and backed with iCloud
- Intelligently shown on lock screen and in search for quick access Continue the seamless Apple Pay experience into the physical world

## Adding passes to Wallet

- instead of using the full modal [`PKAddPassesViewController`][PKAddPassesViewController] to add passes, use [the automatic-pass-adding API][automatic-pass-adding]:

```swift
func addPasses(_
  passes: [PKPass], 
  withCompletionHandler completion: ((PKPassLibraryAddPassesStatus) -> Void)? = nil
)

// With Swift 5.5 concurrency:
func addPasses(_ passes: [PKPass]) async -> PKPassLibraryAddPassesStatus
```

- Presents a simple alert to the user requesting to add or review the passes
- Less friction than presenting `PKAddPassesViewController`
- Lets you handle the completion outcome in the callback

### Best practices

- Suggest adding passes that were created outside of your app 
- Add related passes to Wallet as a group
- Make it easy for people to quickly add passes they do not have 
- Let people jump to their passes in Wallet from your app
- New

## Designing passes

- Use pass fields to display relevant text
- Use vibrant colors to make your pass stand out 
- Design a pass that looks great on all devices 
- Avoid reproducing existing physical passes
- Don’t encode user information in the strip image (not shown on Apple Watch)

### Passes on Apple Watch

- Does not support the strip image 
- Thumbnail image is not displayed 
- Users cannot access the pass details

### Additional row support

- New this year
- can only be used in auxiliary fields in an `eventTicket` pass type
- Can add only one extra row
- Auxiliary fields are then displayed on one row up to the limit of fourth fields per row

```json
"auxiliaryFields": [
  {
    "label": "Date",
    "key": "Date",
    "value": "June 9, 2018",
    "row": 0
  }, {
    "label": "Section",
    "key": "Section",
    "value": "10",
    "row": 1
  }
]
```

- Only values of `0` and `1` are supported
- On older versions, `"row"` is ignored

## Rich pass content

### Relevancy

- Add this functionality with the `locations`, `relevantText`, and `relevantDate` pass JSON fields
- Pass appears on lock screen at the right moment 
- Handles multiple relevant passes
- Always add relevancy information and trust the system to present as required

### Semantic Tags (new)

- Great way to add machine readable information to passes
- [70+ event and transit type tags supported](https://developer.apple.com/documentation/walletpasses/semantictags)
- Relevancy information works in combination with semantic tags
- [Documentation here](https://developer.apple.com/documentation/passkit/wallet/supporting_semantic_tags_in_wallet_passes)

#### Example

You can add additional related data that is not important for display, but associated with a field.

```json
{
  "key": "event",
  "label": "Apple Park",
  "value": "Revenge Of The Passes",
  "semantics": {
    "eventName": "Revenge Of The Passes: A Wallet Story",
    "venueName": "Apple Park 1",
    "venuePhoneNumber": "+1(408)888-8888"
  }
}
```

Semantics can be associated with the pass and not with any field, in the following example we tell PassKit that the phone should be muted during the movie (movie ticket example):

```json
{
  "semantics": {
    "eventType": "PKEventTypeMovie",
    "silenceRequested": true,
    "duration": 7245
  }
}
```

Siri will then be able to offer the user the ability to quickly enable do not disturb at the right time

## Contactless passes

- Requires an NFC Certificate to get started
- [Apply for access](https://developer.apple.com/contact/passkit)
- Your readers must support the Apple Value added Services protocol

[PKAddPassesViewController]: https://developer.apple.com/documentation/passkit/pkaddpassesviewcontroller
[automatic-pass-adding]: https://developer.apple.com/documentation/passkit/pkpasslibrary/1617093-addpasses


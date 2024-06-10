# Embedding and Sharing Visually Rich Links

The new Link Presentation framework enables app developers to easily present URLs in a rich, beautiful, and consistent way. Learn how to use Link Presentation to retrieve metadata from a URL, present the rich link content inside your app, and provide link metadata to the new share sheet experience in iOS.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/262", purpose: link, label: "Watch Video (6 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



## Introduction
In iOS 10 and macOS Sierra, Messages got Rich Links to make URLs more beautiful and useful. Rich links were customized for certain types of media, including audio/video playback, Twitter tweets, Apple Maps links, and more.

iOS 13 and macOS 10.15 introduce new APIs that allow developers to put Rich Links in their own apps.

The video shows how to turn a table view of recipe links into Rich Links. Our starting point looks like this:

![][starting_point]

### Retrieving metadata
![][lp_metadata_diagram]

The `LPMetadataProvider` class takes a URL and returns an `LPLinkMetadata` object with the title and any available media.

Let's assume we're configuring a `UITableViewCell`:
```swift
let metadataProvider = LPMetadataProvider()
let url = URL(string: "https://www.apple.com/ipad")!

metadataProvider.startFetchingMetadata(for: url) { metadata, error in
  if error != nil {
    // The fetch failed; handle the error.
    // Examples: server doesn't respond, is too slow, user doesn't have network.
    return
  }

  // Make use of the fetched metadata.
}
```

#### Things to keep in mind with `LPMetadataProvider` and `LPLinkMetadata`.
- Title, icon, image, and video are all optional.
- `LPLinkMetadata` is serializable with `NSSecureCoding`. This means you can and should cache metadata locally as much as possible.
- Can also be used for local file URLs. Quick Look provides the thumbnail for the file if possible.

### Presenting links
![][lp_link_view]

Continuing the previous example, replace

```swift
// Make use of the fetched metadata
```

with the following

```swift
let linkView = LPLinkView(metadata: metadata)
cell.contentView.addSubview(linkView)
linkView.sizeToFit()
```

Our app now looks like this:

![][rich_links]

Note that `LPLinkView` has an intrinsic size, but it also responds to `sizeThatFits()` for an ideal size given the surrounding constraints, and it will try to present a layout that's reasonable at any size.


### Accelerating the iOS 13 share sheet

For existing apps that share URLs, the share sheet will automatically present a preview of the link at the top of the card.

![][link_at_top]

However, this requires reaching out to the server to retrieve metadata *once the share sheet is presented*, so the title and icon show up asynchronously.

If you already have the metadata, you can send it to the share sheet and the preview will appear instantly:

```swift
// Provide prefetched metadata to UIActivityViewController in your implementation
// of UIActivityItemSource

func activityViewControllerLinkMetadata(_: UIActivityViewController) -> LPLinkMetadata? {
  return self.metadata
}
```

Also, when the user chooses to pass that metadata on, say by sending a message, that preview appears instantly.

Lastly, you can provide custom or missing metadata information inside the `activityViewControllerLinkMetadata(_:)` method:

```swift
func activityViewControllerLinkMetadata(_: UIActivityViewController) -> LPLinkMetadata? {
  let metadata = LPLinkMetadata()

  // `originalURL` and `url` fields are required. Everything else is optional.
  metadata.originalURL = URL(string: "https://www.example.com/apple-pie")
  metadata.url = metadata.originalURL
  metadata.title = "The Greatest Apple Pie In The World"
  metadata.imageProvider = NSItemProvider.init(
    contentsOf: Bundle.main.url(forResource: "apple-pie", withExtension: "jpg")
  )
  return metadata
}
```

### Summary
![][summary]


[starting_point]: WWDC19-262-starting_point

[lp_metadata_diagram]: WWDC19-262-lp_metadata_diagram

[lp_link_view]: WWDC19-262-lp_link_view

[rich_links]: WWDC19-262-rich_links

[link_at_top]: WWDC19-262-link_at_top

[summary]: WWDC19-262-summary

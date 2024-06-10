# Make blazing fast lists and collection views

Build consistently smooth scrolling list and collection views: Explore the lifecycle of a cell and learn how to apply that knowledge to eliminate rough scrolling and missed frames. We’ll also show you how to improve your overall scrolling experience and avoid costly hitches, with optimized image loading and automatic cell prefetching.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10252", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Session code sample](https://developer.apple.com/documentation/uikit/uiimage/building_high-performance_lists_and_collection_views)

## Performance fundamentals 

- Diffable data source is built to store identifiers of items in your model, and not the model objects themselves

Improvements to diffable data source:

- Before iOS 15, applying a snapshot without animation would do a `reloadData` internally
- From iOS 15 onwards, applying a snapshot without animation will only apply the differences and not perform any extra work
- diffable data source also gains a new `reconfigureItems` method that makes it easy to update the contents of visible cells

### Life of a cell 

Two phases:

1. preparation
  - the collection view is asked to dequeue a new cell using a registration
  - if a cell exists in the reuse pool, `UICollectionView` will call `prepareForReuse` on it, and then dequeue the cell. Otherwise it will create a new one
  - the cell is then passed in to the configuration handler from the registration
  - the collection view queries the cell for its preferred layout attributes and sizes the cell appropriately

2. display
  - `willDisplayCell` is called on the delegate
  - the cell is made visible inside the `UICollectionView`
  - when the cell is scrolled off screen, `didEndDisplaying` is called for the cell, and it ends up in the reuse pool

### How an app updates the display

How an app produces a frame:

1. commit

  - for each frame, events such as touches are delivered to the app
  - in response, the app updates the properties of its views and layers
  - as a result of those changes, the app's views and layers perform layout

2. rendering

  - the layer tree is sent to the render server
  - each frame has a commit deadline, this is the time by which all commits for that frame need to finish
  - the amount of time an app has to commit for each frame depends on the refresh rate of the display

What is a hitch?

- when the commit for a frame takes too long and misses the deadline, those updates don't get incorporated into the intended frame
- the display keeps the previous frame on screen until the commit finishes, and this delayed frame can render
- this is a commit hitch and is perceived as a momentary interruption when scrolling

## Cell prefetching

- from iOS 15 cells are pre-fetched after finishing a short commit
- this makes them pre-ready for when they're needed

How prefetching effects the cell lifecycle:

- it is possible for a prepared cell to never be displayed, which could happen if the user suddenly changed the scroll direction
- once a cell is displayed, it can go right back into the waiting state after it goes off screen
- the same cell can be displayed more than once for the same index path

This means that a cell will not be immediately added to the reuse pool when it ends displaying.

## Updating cell content

When a cell needs to get/display remote data, we shouldn't update its content directly, as the cell might have been already reused to display other content.

- iOS 15 introduces the `reconfigureItems` snapshot method: this method will rerun the associated cells registration's configuration handler. 

```swift
private func setPostNeedsUpdate(id: DestinationPost.ID) {
  var snapshot = dataSource.snapshot()
  snapshot.reconfigureItems([id]) // 👈🏻
  dataSource.apply(snapshot, animatingDifferences: true)
}
```

- use this instead of `reloadItems`, because it reuses the item's existing cell, rather than dequeuing and configuring new cells 

Example:

```swift
// Updating cells asynchronously

let cellRegistration = UICollectionView.CellRegistration<DestinationPostCell,
                                                         DestinationPost.ID> {
  (cell, indexPath, postID) in

  let post = self.postsStore.fetchByID(postID)
  let asset = self.assetsStore.fetchByID(post.assetID)
  
  if asset.isPlaceholder {
    self.assetsStore.downloadAsset(post.assetID) { _ in
      self.setPostNeedsUpdate(id: post.id) // 👈🏻
    }
  }
  
  cell.titleView.text = post.region
  cell.imageView.image = asset.image
}
```

- to maximize prepare time, trigger downloads during the data-source prefetching.

### Image preparation APIs

- decoding and displaying images takes time
- ideally, we could prepare the image in advance and only update the UI when the image decoding is completed
- new in iOS 15, we have image preparation APIs, giving us control over where and when image preparation happens
- these APIs produce a new `UIImage`, which only contains the pixel data that the renderer needs
- two forms: 
  - synchronous, which can on any thread
  - asynchronous, which run on an internal UIKit serial queue

```swift
// Using prepareForDisplay

// Initialize the full image
let fullImage = UIImage()

// Set a placeholder before preparation
imageView.image = placeholderImage

// Prepare the full image (in the background)
fullImage.prepareForDisplay { preparedImage in
  DispatchQueue.main.async {
    self.imageView.image = preparedImage
  }
}
```

Warning:

- these prepared images are not to be stored on disk, but should be cached
- they take large amount of memory

Usage example:

```swift
// Asset downloading – with image preparation

func downloadAsset(
  _ id: Asset.ID,
 completionHandler: @escaping (Asset) -> Void
) -> Cancellable {
  // Check for an already prepared image
  if let preparedAsset = imageCache.fetchByID(id) {
      completionHandler(preparedAsset)
      return AnyCancellable {}
  }
  return fetchAssetFromServer(assetID: id) { asset in
    asset.image.prepareForDisplay { preparedImage in
      // Store the image in the cache.
      self.imageCache.add(asset: asset.withImage(preparedImage!))
      DispatchQueue.main.async {
        completionHandler(asset)
      }
    }
  }
}
```

New thumbnail API:

- use less memory and computation

```swift
// Using prepareThumbnail

// Initialize the full image
let profileImage = UIImage(...)

// Set a placeholder before preparation
posterAvatarView.image = placeholderImage

// Prepare the image
profileImage.prepareThumbnail(of: posterAvatarView.bounds.size) { thumbnailImage in
  DispatchQueue.main.async {
    self.posterAvatarView.image = thumbnailImage
  }
}
```

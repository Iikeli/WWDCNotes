# A Tour of UICollectionView

UICollectionView is a flexible, powerful tool to help you achieve great user experiences in your applications. Hear how you can leverage these rich APIs to rapidly move from initial design ideas to polished shipping applications. Topics range from getting started to advanced update animations and layouts.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/225", purpose: link, label: "Watch Video (40 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Code sample](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/layouts/customizing_collection_view_layouts)

## Key UICollectionView Concepts

- Layout
- Data source
- Delegate

### Layout

[`UICollectionViewLayout`][UICollectionViewLayout]:

- allows `UICollectionView` to abstract away the visual arrangement of your content from the content itself
- is all about the _where_ content is displayed
- supports invalidation for when, for example, you want to change the appearance of the layout
- can animate between layouts (without the need for layouts knowing anything about other layouts)
- `UICollectionViewLayout` is an abstract class not meant to be used directly. Subclasses of `UICollectionViewLayout`, such as [`UICollectionViewFlowLayout`][UICollectionViewFlowLayout], are meant to be used

Each individual item is specified by [`UICollectionViewLayoutAttributes`][UICollectionViewLayoutAttributes]:

- contains attributes for visual arrangement such as bounds, center, and frame
- you can think of it as a set of properties you can use to define these items that are displayed

#### `UICollectionViewFlowLayout`

- line-based layout covers a wide range of designs
- line spacing: space between lines (you can set the min space via `minimumLineSpacing`)
- inter-item spacing: space within items in the same line (you can set the min space via `minimumInteritemSpacing`)

###  Data source

Layout is all about the _where_ content goes, the data source is the _what_

[`UICollectionViewDataSource`][UICollectionViewDataSource] has three core methods:

```swift
/// Defaults to one section when not implemented
optional func numberOfSections(in collectionView: UICollectionView) -> Int

/// Number of items for the given section
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int

/// Actual content you're going to display to your users
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell
```

### Delegate

[`UICollectionViewDelegate`][UICollectionViewDelegate]:

- Optional to implement
- Extends `UIScrollViewDelegate`
- Fine-grained control over:
  - Highlighting
  - Selection

- View appearance events
  - `willDisplayItem`
  - `didEndDisplayingItem`
 
## `UICollectionViewLayout`

[`prepare()`][prepare()] is called whenever the layout is invalidated. In the case of `UICollectionFlowLayout`, our layout is invalidated whenever the `UICollectionView`'s `bounds`'s size changes. This is a great place to do any customization that takes the size of the `UICollectionView` into account. 

In the following example we use `prepare()` to update the `itemSize` and sections insets based on the `collectionView.bounds`:

```swift
class ColumnFlowLayout: UICollectionViewFlowLayout {
  private let minColumnWidth: CGFloat = 300.0
  private let cellHeight: CGFloat = 70.0

  override func prepare() {
    super.prepare()
    guard let collectionView = collectionView else { return }
	  
    let availableWidth = collectionView.bounds.inset(by: collectionView.layoutMargins).width
    let maxNumColumns = Int(availableWidth / minColumnWidth)
    let cellWidth = (availableWidth / CGFloat(maxNumColumns)).rounded(.down)
	  
    self.itemSize = CGSize(width: cellWidth, height: cellHeight)
    self.sectionInset = UIEdgeInsets(top: self.minimumInteritemSpacing, left: 0.0, bottom: 0.0, right: 0.0)
    self.sectionInsetReference = .fromSafeArea
  }
  ...
}
```

## Creating a Custom `UICollectionViewLayout`

We should try to stick with `UICollectionFlowLayout` whenever possible, however, if our layout is not line-based, it's completely acceptable to subclass `UICollectionViewLayout`.

Four basic methods

### 1. Providing Content Size

```swift
open var collectionViewContentSize: CGSize { get }
```

- Size of bounds which contains all items - a.k.a., if you imagine a rectangle that encompassed all the content that the layout is going to define for your `UICollectionView`, `UICollectionViewLayout` needs to return the size of that.
- Needed for `UIScrollView.contentSize`

### 2. & 3. Providing Layout Attributes

```swift
/// Called periodically by `UICollectionView` when it needs to know what is needed to display 
/// on screen as the user scrolls through your content, or displays for the first time.
///
/// In other words `UICollectionView` is asking for a set of attributes that match a certain 
/// region. It's our job to return an array that contains all the attributes that correspond to all
/// the items that are going to appear within that rect in our `UICollectionView`.
func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]?

/// Asks for the `UICollectionViewLayoutAttributes` of the given item.
func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes?
```

- Query by geometric region
- Query by IndexPath
- Performance matters

### 4. Preparing the Layout

```swift
func prepare()
```

- Called for every `invalidateLayout()` (just once per invalidation)
- This is the time for you to:
  - cache `UICollectionViewLayoutAttributes`
  - compute `collectionViewContentSize`

```swift
class CustomLayout: UICollectionViewLayout {
  // Cached information
  var contentBounds: CGRect = .zero
  var cachedAttributes: [UICollectionViewLayoutAttributes] = []
  
  override func prepare() {
      super.prepare()
      
      guard let collectionView = collectionView else { return }

      // Reset cached information.
      cachedAttributes.removeAll()
      contentBounds = CGRect(origin: .zero, size: collectionView.bounds.size)
      
      // For every item in the collection view:
      //  - Prepare the attributes.
      //  - Store attributes in the cachedAttributes array.
      //  - Combine contentBounds with attributes.frame.
      ...
    }
  }
}
```

### 5. (bonus) Handling Bounds Changes in Your Custom Layout

```swift
func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool
```

- Called for every `bounds` change (both `size` and/or `origin`)
- Called during scrolling (as the origin changes)
- Default implementation returns `false`
- `UICollectionFlowLayout` returns `true` only when `bounds.size` changes

## `performBatchUpdates(_:completion:)`

[`performBatchUpdates(_:completion:)`][performBatchUpdates(_:completion:)]:

- Animate updates together
- Perform data source updates and collection view updates in updates closure
- Collection view updates ordering does not matter within the block
- data source updates ordering does matter

Collection View Updates Coalescing

| Action | Notes | Index Path Semantics |
| --- | --- | --- |
| Delete | Descending IndexPath order | Before updates |
| Insert | Ascending IndexPath order | After updates |
| Move |  | From: Before updates. To: After updates. |
| Reload | Decompose: Delete and inserts | Before updates |
 
Exceptions will result from:

- Move and Delete the same location
- Move and Insert to the same location
- Move more than 1 location to the same location
- Referencing an invalid IndexPath

How to use `performBatchUpdates(_:completion:)` safely:

- Decompose Move into Delete and Insert updates
- Combine all Delete and Insert updates
- Process Delete updates first, in descending order
- Process Insert updates last, in ascending order

[performBatchUpdates(_:completion:)]: https://developer.apple.com/documentation/uikit/uicollectionview/1618045-performbatchupdates
[prepare()]: https://developer.apple.com/documentation/uikit/uicollectionviewlayout/1617752-prepare
[UICollectionViewDelegate]: https://developer.apple.com/documentation/uikit/uicollectionviewdelegate
[UICollectionViewDataSource]: https://developer.apple.com/documentation/uikit/uicollectionviewdatasource
[UICollectionViewFlowLayout]: https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout
[UICollectionViewLayoutAttributes]: https://developer.apple.com/documentation/uikit/uicollectionviewlayoutattributes
[UICollectionViewLayout]: https://developer.apple.com/documentation/uikit/uicollectionviewlayout

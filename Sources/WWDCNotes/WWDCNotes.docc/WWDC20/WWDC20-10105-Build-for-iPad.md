# Build for iPad

Learn how to improve iPad apps to leverage the increased screen size and additional features of iPadOS, and help people accomplish more with their devices. Discover how you can build detailed multi-column layouts and integrate lists into your app with little adjustment to your existing code. We’ll also explore reducing modality within your views to make it easier to navigate your interface with fewer taps and touches.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10105", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Multi-column Split View

The core of a multi-column app is [`UISplitViewController`][UISplitViewController].

### Two-Column layout

![][doubleImage]

Define your split view controller:

```swift
let splitViewController = UISplitViewController(style: .doubleColumn)
```

Set the view for each column:

```swift
splitViewController.setViewController(sidebarViewController, for: .primary)
splitViewController.setViewController(myHomeViewController, for: .secondary)
```

### Three-column layout

![][tripleImage]

Define your split view controller:

```swift
let splitViewController = UISplitViewController(style: .tripleColumn)
```

### Behind the scenes

Apple wants you to use this so your app adapts to both iPad and iPhone:

- `UISplitViewController` handles the columns for you by showing the right view controllers based on size classes.
- we can specify a different view to be used when in compact mode via the `UISplitViewController`'s [`setViewController(_:for:)`][setViewController(_:for:)] method and passing `.compact` as the column layout. This view can have a completely different navigation (for example a tab bar).

### Display Modes

- The different ways `UISplitViewController` can lay out your columns are called [`DisplayModes`][DisplayModes]: they differ depending on how much space we have and how big they are.

- To move between these display modes, `UISplitViewController` automatically creates appropriate buttons and makes them appear in the right places.

- Some display modes can be shown via gestures, if needed, disable them via the [`presentsWithGesture`][presentsWithGesture] property.

- Use [`showsSecondaryOnlyButton`][showsSecondaryOnlyButton] to enable an additional button which hides all but the secondary column.

#### Double Column Display Modes

![][doubleDisplayImage]

#### Triple Column Display Modes

![][tripleDisplayImage]

#### Setting the preferred split behavior

Set your app preferred split behavior via [`preferredSplitBehavior`][preferredSplitBehavior]:

`.tile`:

![][tileImage]

`.displace`:

![][displaceImage]

`.overlay`:

![][overlayImage]

### Hide Columns

At any time we can hide any of the columns:

```swift
splitViewController.hideColumn(.primary)

splitViewController.showColumn(.supplementary)
```

### Navigation Controllers

![][navigationImage]

Each `UISplitViewController` column has a navigation controller that's automatically created, and each navigation controller has a navigation bar at the top and an optional toolbar at the bottom.

## Lists

Setup your primary/secondary views by building a list via `UICollectionView`:

```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .sidebar)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
```

Then register your cells with the new [`CellRegistration`][CellRegistration] API:

```swift
let cellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, MyItem> { cell, indexPath, item in

    var content = cell.defaultContentConfiguration()

    content.text = item.title
    content.image = item.image

    cell.contentConfiguration = content
}
```

And create a diffable data source: 

```swift
let dataSource = UICollectionViewDiffableDataSource<Section, MyItem>(
        collectionView: collectionView
    ) { collectionView, indexPath, item in
   collectionView.dequeueConfiguredReusableCell(using: cellRegistration, 
                                                for: indexPath,
                                                item: item)
}
```

In the supplementary column do the same but with the plain style:

```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .sidebarPlain)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
let collectionView = UICollectionView(frame: frame, collectionViewLayout: layout)
```

[UISplitViewController]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller
[setViewController(_:for:)]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller/3580911-setviewcontroller
[presentsWithGesture]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller/1623171-presentswithgesture
[showsSecondaryOnlyButton]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller/3580913-showssecondaryonlybutton
[DisplayModes]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller/displaymode
[preferredDisplayMode]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller/1623170-preferreddisplaymode
[CellRegistration]: https://developer.apple.com/documentation/uikit/uicollectionview/cellregistration
[preferredSplitBehavior]: https://developer.apple.com/documentation/uikit/uisplitviewcontroller/3580909-preferredsplitbehavior

[doubleImage]: WWDC20-10105-double
[tripleImage]: WWDC20-10105-triple
[doubleDisplayImage]: WWDC20-10105-doubleDisplay
[tripleDisplayImage]: WWDC20-10105-tripleDisplay
[tileImage]: WWDC20-10105-tile
[displaceImage]: WWDC20-10105-displace
[overlayImage]: WWDC20-10105-overlay
[navigationImage]: WWDC20-10105-navigation

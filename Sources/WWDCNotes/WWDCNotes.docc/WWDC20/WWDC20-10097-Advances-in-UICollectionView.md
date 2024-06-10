# Advances in UICollectionView

Learn about new features of UICollectionView that make it easier to use and unlock powerful new functionality. We'll show you how to use section snapshots with your diffable data source to create outlines that can expand and collapse, and introduce you to building lists with compositional layout to create UITableView-like interfaces with a collection view. And discover modern techniques for dequeuing cells and configuring their content and styling.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10097", purpose: link, label: "Watch Video (9 min)")

   @Contributors {
      @GitHubUser(dasautoooo)
   }
}


## Sample Code
Download the [sample code](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views) to learn more about Modern Collection Views.

## Modern Collection View APIs Overview
![][modern_collection_view_apis]

## Diffable Data Source
### Recap

* Introduced in iOS 13.
* Simplifies UI State.
* Automatic animations.
* More to watch session [220](https://developer.apple.com/videos/play/wwdc2019/220) from WWDC 19: Advances in UI Data Sources.

### New in iOS 14
#### [Section Snapshots](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesectionsnapshot)
* Encapsulates the data for a single section in a UICollectionView.
* Allow data sources to be more composable into section-sized chunks of data.
* Can present the same data with different layout in different sections.
* Allow modeling of hierarchical data, which is needed to support rendering outline-style UIs.

#### Examples
![][section_snapshots]

* Top: Horizontally scrolling section.
* Middle: Outline style section.
* Bottom: List section.

For more refer to the [`Advances in diffable data sources`][wwdc2010045] session.

## Compositional Layout
### Recap
* Introduced in iOS 13.
* Allows us to build rich, complex layouts by composing smaller, easy-to-reason bits of layout together.
* Describes what we want the layout to look like instead of how the layout ought to work.
* Provides section-specific layouts.
* For more refer to the [`Advances in Collection View Layout`][wwdc19215] session.

### New in iOS 14
#### [Lists](https://developer.apple.com/documentation/uikit/uicollectionviewcompositionallayout)
* `UITableView`-like appearance.
* Rich with features like swipe actions and many common cell layouts.
* Built on top of Compositional Layout.
* New concrete UICollectionViewListCell.
* Header and footer support.
* Sidebar appearance.

#### Examples
Creating a `UICollectionViewLayout` where every section in the collection view is a list with an inset grouped appearance.

```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

More to watch session [10026](https://developer.apple.com/wwdc20/10026): Lists in UICollectionView.

## Modern Cells
### [Cell Registrations](https://developer.apple.com/documentation/uikit/uicollectionview/cellregistration)
* Brand new way to configure cells.
* Simple, re-usable way to set up a cell from a view model.
* Eliminates the extra step of registering a cell class or nib to associate it with a re-use identifier.
* Uses a generic registration type, which incorporates a configuration closure for setting up a new cell from a view model.

```swift
let reg = UICollectionView.CellRegistration<MyCell, ViewModel> { cell, indexPath, model in
   // configure cell content 
}

let dataSource = UICollectionViewDiffableDataSource<S,I>(collectionView: collectionView) {
   collectionView, indexPath, item -> UICollectionViewCell in
   return collectionView.dequeueConfiguredReusableCell(using: reg, for: indexPath, item: item)
}
```

### [Cell Content Configurations](https://developer.apple.com/documentation/uikit/uilistcontentconfiguration)
* Similar to `UITableViewCell`.
* To be used as any cell or even a generic UI view.
* Flexible.
* Lightweight.

```swift
var contentConfiguration = UIListContentConfiguration.cell()
contentConfiguration.image = UIImage(systemNamed:"hammer")
contentConfiguration.text = "Ready. Set. Code"
cell.contentConfiguration = contentConfiguration
```

### [Background Configurations](https://developer.apple.com/documentation/uikit/uibackgroundconfiguration)
* Similar to content configurations, but applied to any cell's background with the ability to adjust properties such as color, border styles, and more.

```swift
cell.backgroundConfiguration = UIBackgroundConfiguration.clear()
```

More to watch session [10027](https://developer.apple.com/wwdc20/10027): Modern cell configuration.

[wwdc19215]: ../../wwdc19/215
[wwdc2010045]: ../10045
[modern_collection_view_apis]: modern_collection_view_apis.png
[section_snapshots]: section_snapshots.png

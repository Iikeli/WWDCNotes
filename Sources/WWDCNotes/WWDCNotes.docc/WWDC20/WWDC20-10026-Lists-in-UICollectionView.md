# Lists in UICollectionView

Learn how to build lists and sidebars in your app with UICollectionView. Replace table view appearance while taking advantage of the full flexibility of compositional layout. Explore modular layout options and find out how they can unlock more design options for your apps than ever before. Find out how to combine table view-like lists with custom compositional layouts inside of a single UICollectionView. Discover how to work with lists, create richer cells, and customize your layout to create a well-designed presentation of information within your app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10026", purpose: link, label: "Watch Video (16 min)")

   @Contributors {
      @GitHubUser(dasautoooo)
   }
}


## Sample Code
Download the [sample code](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views) to learn more about Modern Collection Views.

## [Lists](https://developer.apple.com/documentation/uikit/uicollectionviewcompositionallayout)

* `UITableView` like appearance.
* Based on Compositional Layout.
* Highly customizable.
* Improved self-sizing support. (No longer need to worry about manually calculating the height of cells)

## Components of a list
![][components_list]

* `UICollectionLayoutListConfiguration` is the only new type required to build a `list`.
* `UICollectionLayoutListConfiguration` is built on top of `NSCollectionLayoutSection` as well as `UICollectionViewCompositionalLayout`.
* For more refer to the [Advances in Collection View Layout](../../wwdc19/215) session.

## [List Configuration](https://developer.apple.com/documentation/uikit/uicollectionlayoutlistconfiguration)
* Gives the styles form table view such as `.plain`, `.grouped` and `.insetGrouped`.
* Two new styles exclusive to lists in `UICollectionView`: `.sidebar` and `.sidebarPlain`.

## Build a `list`
### Simple setup
```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

1. Create an `UICollectionLayoutListConfiguration`, give it an appearance.
2. Create an `UICollectionViewCompositionalLayout` using this configuration.

### Per section setup
```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)
```

1. Create an `UICollectionLayoutListConfiguration`, give it an appearance.
2. Create an `NSCollectionLayoutSection` using this configuration.

This code can then be used inside the existing section provider initializer on compositional layout.

```swift
let layout = UICollectionViewCompositionalLayout() {
    [weak self] sectionIndex, layoutEnvironment in
    guard let self = self else { return nil }

    // @todo: add custom layout sections for various sections
  
    let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)
    return section
}
``` 

## Headers and Footers
Headers and footers in UICollectionView have to be explicitly enabled.

### One approach
Works for both Headers and Footers.

#### First
* Register headers and footers as supplementary views.

```swift
var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.headerMode = .supplementary
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```

#### Second
* Provide this view through a `supplementaryViewProvider` on your diffable data source.
*  Can also implement the equivalent method on UICollectionView delegate.
* Check the `elementKind` for either `elementKindSectionHeader` or `elementKindSectionFooter` and configure and return the appropriate view.

```swift
var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.headerMode = .supplementary
let layout = UICollectionViewCompositionalLayout.list(using: configuration)

dataSource.supplementaryViewProvider = { (collectionView, elementKind, indexPath) in
    if elementKind == UICollectionView.elementKindSectionHeader {
        return collectionView.dequeueConfiguredReusableSupplementary(using: header, for: indexPath)
    }
    else {
        return nil
    }
}
```

* Set the mode to either `.supplementary` or `.none` depending on whether this particular section should show a header or not. 
* Don't return nil in the `supplementaryViewProvider`.

```swift
let configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.headerMode = sectionHasHeader ? .supplementary : .none
let section = NSCollectionLayoutSection.list(using: configuration, layoutEnvironment: layoutEnvironment)
```

### Another Approach
Only available for headers.

```swift
var configuration = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
configuration.headerMode = .firstItemInSection
let layout = UICollectionViewCompositionalLayout.list(using: configuration)
```
* Sets the `headerMode` to `.firstItemInSection`. 
* Collection view will configure the first cell to look like a header.
* Data source needs to be aware of this.
* The first item in data source no longer represents the actual content of section but rather the header.

## [List Cell](https://developer.apple.com/documentation/uikit/uicollectionviewlistcell)
Can use either `UICollectionViewListCell` or `UICollectionViewCell` in lists sections.

* Support to configure the insets of separators
* Support to configure the indentation of cell's content.
* Swipe Actions are now also a feature of the cell.
* Improved accessories API.
* Access to the default system content and background configurations.
* For more refer to the [Modern cell configuration](https://developer.apple.com/videos/play/wwdc2020/10027) session.

### Separators
![][wrong_separator]
![][right_separator]

* Separator is supposed to line up with the primary content of cell.
* Use [Separator Layout Guide](https://developer.apple.com/documentation/uikit/uicollectionviewlistcell/3601206-separatorlayoutguide) to easily align the separator.
* Separator Layout Guide works opposite of other layout guides. 
* Constrain this layout guide to content.
* Configure cell's layout first, then constrain the separator layout guides leading anchor to primary content.

### Swipe Actions

* Swipe actions are now a feature of list cell.
* Configure like cell content.
* Swipe Actions are only supported using a list configuration.

```swift
let markFavorite = UIContextualAction(style: .normal, title: "Mark as Favorite") {
	[weak self] (_, _, completion) in
	guard let self = self else { return }
	// trigger the action with a reference to the model
	self.markItemAsFavorite(with: item.identifier)
	completion(true)
}
cell.leadingSwipeActionsConfiguration = UISwipeActionsConfiguration(actions: [markFavorite])
```

🚨**Caution**:

* Make sure to never capture the index path of the cell.
* The index path is not a stable identifier.
* The index path of this cell changes whenever we're inserting or deleting content above it which doesn't necessarily reload this particular cell.
* Capture the data model directly or a stable identifier instead.

### [Accessories](https://developer.apple.com/documentation/uikit/uicollectionviewlistcell/3601206-separatorlayoutguide)

* List cell offers many new accessory types.
* Allows to configure accessories for both, the trailing and the leading side of the cell.
* Can configure multiple accessories on the same side.

```swift                                                   
cell.accessories = [ 
    .checkmark(), 
    .disclosureIndicator(options: .init(tintColor: .systemGray)), 
    .delete(),
    .reorder() 
]
```

[components_list]: components_list.png
[wrong_separator]: wrong_separator.png
[right_separator]: right_separator.png

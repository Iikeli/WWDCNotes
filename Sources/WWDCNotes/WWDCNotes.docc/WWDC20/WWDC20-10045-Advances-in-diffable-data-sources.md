# Advances in diffable data sources

Diffable data sources dramatically simplify the work involved in managing and updating collection and table views to create dynamic and responsive experiences in your apps. Discover how you can use section snapshots to efficiently build lists and outline collection views for iOS and iPadOS and provide support for implementing the sidebar in an iPad app. We’ll also show you how to simplify cell reordering using UICollectionViewDiffableDataSource to help you streamline your code and build app interfaces more quickly.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10045", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(dasautoooo)
   }
}



## Sample Code

Download the [sample code](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views) to learn more about Modern Collection Views.

## Diffable Data Source

### Recap

* Introduced in iOS 13.
* Simplifies UI State.
* Automatic animations.
* No more batch updates.
* For more, refer to the [`Advances in UI Data Sources` session][wwdc19220].

### [Section Snapshots](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesectionsnapshot)
* Encapsulate the data for a single section in a `UICollectionView`.
* Allow data sources to be more composable into section-sized chunks of data.
* Can present the same data with different layout in different sections.
* Allow modeling of hierarchical data, which is needed to support rendering outline-style UIs.

#### Examples
![][section_snapshots]

* Top: Horizontally scrolling section.
* Middle: Outline style section.
* Bottom: List section.

It composes Diffable Data Source from three distinct section snapshots, each representing a single section's content.

#### APIs

* Use `append(:parent:)` API to add content into a section snapshot.
* The optional `parent` parameter allows us to create parent-child relationships in the section snapshot.

```swift
// UICollectionViewDiffableDataSource additions for iOS 14

extension UICollectionViewDiffableDataSource<Item, Section> {

    func apply(_ snapshot: NSDiffableDataSourceSectionSnapshot<Item>, 
               to section: Section, 
               animatingDifferences: Bool = true, 
               completion: (() -> Void)? = nil)

    func snapshot(for section: Section) ->   
                  NSDiffableDataSourceSectionSnapshot<Item>
}
```

Code example:

```swift
// Example of using snapshots and section snapshots together

func update(animated: Bool=true) {

   // Add our sections in a specific order
   let sections: [Section] = [.recent, .top, .suggested]
   var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
   snapshot.appendSections(sections)
   dataSource.apply(snapshot, animatingDifferences: animated)

   // update each section's data via section snapshots in the existing position
   for section in sections {
      let sectionItems = items(for: section)
      var sectionSnapshot = NSDiffableDataSourceSectionSnapshot<Item>()
      sectionSnapshot.append(sectionItems)
      dataSource.apply(sectionSnapshot, to: section, animatingDifferences:animated)
   }
}
```

#### Child snapshots

```swift
let childSnapshot = sectionSnapshot.snapshot(for: parent, includingParent: false)
```

#### Expanding and Collapsing Items

* Expansion state is persisted. 
* Expansion state is managed as part of a section snapshot's state.
* Call `apply(_:to)` to commit changes.
* Automatic animations.

```swift
struct NSDiffableDataSourceSectionSnapshot<Item: Hashable> {
   func expand(_ items: [Item])
   func collapse(_ items: [Item])
   func isExpanded(_ item: Item) -> Bool
}
```

#### [Section snapshot handlers](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3600966-sectionsnapshothandlers)
To be notified about expansion state changes caused by these user interactions. 

```swift
// Section Snapshot Handlers: handling user interactions for expand / collapse state changes

extension UICollectionViewDiffableDataSource {

  struct SectionSnapshotHandlers<Item> {
    var shouldExpandItem: ((Item) -> Bool)?
    var willExpandItem: ((Item) -> Void)?
	
    var shouldCollapseItem: ((Item) -> Bool)?
    var willCollapseItem: ((Item) -> Void)?
    
    var snapshotForExpandingParent: ((Item, NSDiffableDataSourceSectionSnapshot<Item>) -> NSDiffableDataSourceSectionSnapshot<Item>)?
  }
  
  var sectionSnapshotHandlers: SectionSnapshotHandlers<Item>
 
}
```

* `SectionSnapshotHandlers` is a generic struct which contains five optional closures.
* `snapshotForExpandingParent` is used for lazy loading when the content is expensive.

### Reordering Support

* Automatic snapshot updates.
* Transactions.

#### [Reordering Handlers](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/reorderinghandlers)

- Used to be notified when a user-initiated reordering interaction took place.
- Once notified, it can persist the new visual order to the application's backing store, which is its final source of truth.

```swift
// Diffable Data Source Reordering Handlers

extension UICollectionViewDiffableDataSource {

  struct ReorderingHandlers {
    var canReorderItem: ((Item) -> Bool)?
    var willReorder: ((NSDiffableDataSourceTransaction<Section, Item>) -> Void)?
    var didReorder: ((NSDiffableDataSourceTransaction<Section, Item>) -> Void)?
  }

  var reorderingHandlers: ReorderingHandlers
}
```

* Automatic reordering.
* When the user is done with the reordering interaction, the `didReorder` closure is called.
* Must provide both the `canReorderItem` and `didReorder` closure to enable the reordering feature.

#### [Reordering Transactions](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcetransaction)
A transaction that describes the changes after reordering the items in the view.

```swift
struct NSDiffableDataSourceTransaction<Section, Item> {
   var initialSnapshot: NSDiffableDataSourceSnapshot<Section, Item> { get }
   var finalSnapshot: NSDiffableDataSourceSnapshot<Section, Item> { get }
   var difference: CollectionDifference<Item> { get }
   var sectionTransactions: [NSDiffableDataSourceSectionTransaction<Section, Item>] { get }
}
```
* `initialSnapshot` is the state of the Diffable Data Source before the update is applied.
* `finalSnapshot` is the state of the Diffable Data Source after the update is applied.
* Apply the data directly to `difference`, if use a data type such as `Array` for the source of truth.
* `sectionTransactions` provides per-section details about all the sections involved in this reordering update.

#### [Section Transactions](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesectiontransaction)
```swift
struct NSDiffableDataSourceSectionTransaction<Section, Item> {
   var sectionIdentifier: Section { get }
   var initialSnapshot: NSDiffableDataSourceSectionSnapshot<Item> { get }
   var finalSnapshot: NSDiffableDataSourceSectionSnapshot<Item> { get }
   var difference: CollectionDifference<Item> { get }
}
```
* `sectionIdentifier` for inspecting which section this `sectionTransaction` has been applied to.

#### Example
```swift
dataSource.reorderingHandlers.didReorder = { [weak self] transaction in 
   guard let self = self else { return }

   if let updateBackingStore = self.backingStore.applying(transaction.difference) {
      self.backingStore = updatedBackingStore
   }
}
```

[wwdc19220]: ../../wwdc19/220
[section_snapshots]: WWDC20-10045-section_snapshots
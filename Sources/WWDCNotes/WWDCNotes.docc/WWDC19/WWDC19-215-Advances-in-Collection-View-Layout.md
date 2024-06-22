# Advances in Collection View Layout

Collection View Layouts make it easy to build rich interactive collections. Learn how to make dynamic and responsive layouts that range in complexity from basic lists to an advanced, multi-dimensional browsing experience.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/215", purpose: link, label: "Watch Video (50 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## `UICollectionViewCompositionalLayout`

With iOS 13 we have a new class `UICollectionViewCompositionalLayout`.

As the name says, its main property is that it is **composable**:  
we have items (think cells) that go inside groups, that go inside sections, that go inside the layout.

![][chartImage]

Every element (Item/group/section) has an explicit size of type [`NSCollectionLayoutDimension`][dimensionDoc], we can have:

- Fractional size: we can say width 25% of its container
- Absolute size: give an exact number
- Estimated: we can just give an estimate and this is flexible

Every element defines it size to referring to its container: if an item says it’s 50% width of its container (group), and the group is 50% of screen width, this means that our item will be 25% of the screen width

We can consider an Item as “_a thing that renders on screen_”, a cell.

We have three forms of [Group][groupDoc] ([`NSCollectionLayoutGroup`][groupDoc]): 

- Vertical (axis)
- Horizontal (axis)
- Custom: lets you specify everything.

A group represent some kind of repeating structure the collection is going to have. 
A column of lists for example.

Lastly the section, `NSCollectionLayoutSection`, defines the layout definition of one section, which means each section can have crazy different layouts.

Since Items are just a concept that we have to fill in, we can now have different types of items (per section!), such us headers, footers, and more.

With `UICollectionViewCompositionalLayout` we can anchor things on an item or at the top/bottom of a section.

We can have nested groups.

The new `UICollectionViewCompositionalLayout` enables to do crazy things that previously would have required tons of custom code/behaviour: no longer.

An example of adoption of this layout class is the App Store home screen, which is now a single collection view controller:
![][appStoreImage]

[dimensionDoc]: https://developer.apple.com/documentation/uikit/nscollectionlayoutdimension
[groupDoc]: https://developer.apple.com/documentation/uikit/nscollectionlayoutgroup?language=objc

[chartImage]: WWDC19-215-chart
[appStoreImage]: WWDC19-215-appStore
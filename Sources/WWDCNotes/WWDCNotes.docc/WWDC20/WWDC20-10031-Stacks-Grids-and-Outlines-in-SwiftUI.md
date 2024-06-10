# Stacks, Grids, and Outlines in SwiftUI

Display detailed data in your SwiftUI apps more quickly and efficiently with improved stacks and new list and outline views. Now available on iOS and iPadOS for the first time, outlines are a new multi-platform tool for expressing hierarchical data that work alongside stacks and lists. Learn how to use new and improved tools in SwiftUI to display more content on screen when using table views, create smooth-scrolling and responsive stacks, and build out list views for content that needs more than a vStack can provide. Take your layout options even further with the new grid view, as well as disclosure groups.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10031", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Stacks

When working with stacks of many elements embedded in `ScrollView`s, instead of loading everything immediately with `HStack`/`VStack`, we can now use [`LazyHStack`][lazyHstack] and [`LazyVStack`][lazyVstack], which load their children lazily.

Not all stacks should be lazy, for example `overlay`s and `background`s are views that will be drawn only when it's time to be displayed, therefore using lazy stacks don't bring any benefit in these cases.

## Grids

SwiftUI Grids are the equivalent of css's [Flexbox layout][flexbox].

[`LazyVGrid`][lazyVGrid] and [`LazyHGrid`][lazyHGrid], similar to stacks, but for multi-column layouts.

When working with grids, we define our columns/rows layout via an array of [`GridItem`][gi] instances.  
Each item will specify how its relative column width/row height is computed.

`GridItem`s are flexible by default, therefore taking all the space available:

```swift
// Creates a three column layout, with equally distributed space between the three columns.

var columns = [
    GridItem(spacing: 0),
    GridItem(spacing: 0),
    GridItem(spacing: 0)
]

ScrollView {
    LazyVGrid(columns: columns, spacing: 0) {
        ForEach(...) { ... in
           ...
        }
    }
}
```

We can also create an adaptive grid layout by declaring

```swift
// Creates as many columns as possible, each with a minimum width of 300 points.

var columns = [
    GridItem(.adaptive(minimum: 300), spacing: 0)
]

ScrollView {
    LazyVGrid(columns: columns, spacing: 0) {
        ForEach(...) { ... in
           ...
        }
    }
}
```

## Progressive Display of Information

### Disclosure Groups

[`DisclosureGroup`][dg] is a new SwiftUI element that shows/hides content to the user based on a boolean `@Binding`.

```swift
DisclosureGroup(isExpanded: $showingContent) {
   // content here
} label: {
   Text("Show content")
}
```

It can be seen as a more adaptive version of the following code:

```swift
Button { 
   showingContent.toggle() 
} label: {
   Text("Show content")
}

if showingContent {
   // content here
}
```

Passing a binding is optional:  
if we don't provide one, the view will use an internal binding and start with `isExpanded` set to `false`.

### Outlines

An [`OutlineGroup`][og] is similar to a `ForEach` except that, instead of iterating over a flat collection, `OutlineGroup` traverses a tree structure data.

For example, this view can be used to display and explore a folder structure or a binary tree.

```swift
OutlineGroup(
    array,
    children: \.children
) { element in
    ...
}
```

`OutlineGroup` needs to know how to traverse our model, therefore it has a few requirements: 

1. we must pass a collection of `Identifiable` objects
2. these objects need to have a property, passed via `\.children` keypath in the example above, that contain an optional collection of the same type

By default the user:

- will only see the elements of the top _"level"_ of the structure at first
- will be able to tap on those elements that contain "sub-collections" to display them.

Behind the scenes `OutlineGroup` uses `DisclosureGroup` to show/hide the nested elements.

`OutlineGroup` is not eager: it computes its rows on demand, based on what it needs to display.

### Lists

`List` has also gained the same capabilities of `OutlineGroup`.

[lazyVstack]: https://developer.apple.com/documentation/swiftui/lazyvstack 
[lazyHstack]: https://developer.apple.com/documentation/swiftui/lazyhstack 
[lazyHGrid]: https://developer.apple.com/documentation/swiftui/lazyhgrid
[lazyVGrid]: https://developer.apple.com/documentation/swiftui/lazyvgrid
[gi]: https://developer.apple.com/documentation/swiftui/griditem
[og]: https://developer.apple.com/documentation/swiftui/outlinegroup
[dg]: https://developer.apple.com/documentation/swiftui/DisclosureGroup
[flexbox]: https://en.wikipedia.org/wiki/CSS_Flexible_Box_Layout
# Mastering Xcode Previews

Xcode 11 displays previews of your user interface right in the editor, streamlining the edit-debug-run cycle into a seamless workflow. Learn how previews work, how to optimize the structure of your SwiftUI app for previews, and how to add preview support to your existing views and view controllers.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/233", purpose: link, label: "Watch Video (44 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- The preview API is part of SwiftUI, but we can use it for preview UIKit views as well.

- To create previews on the current file, create a new `Struct` conforming to the [`PreviewProvider` protocol][previewProviderDoc]:  
usually this preview will returns the view of the current file and that’s it.

- Wrap the preview declarations with `#if DEBUG` `#endif` compile conditions to make sure previews will be compiled only in debug mode.

- We can select the device destination on the preview screen itself, or we can specify it in our preview code by using [`.previewDevice(“iPhone X”)`][previewDeviceDoc].

- Instead of a device, we can also focus on the raw view itself (a-la `xib` files), and that is done via the [`.previewLayout()` API][previewLayoutDoc].  

- Use `Group { .. }` to display multiple previews at once.

- Override environment variables with the `.environment(\.sizeCategory, .extraLarge) API`

- Use [`forEach`][forEachDoc] to test all cases:

```swift
#if DEBUG

struct ContentView_Previews: PreviewProvider {
  static var previews: some View {
    Group {
      ForEach(ContentSizeCategory.allCases, id: \.hashValue) { size in
        ContentView()
          .environment(\.sizeCategory, size)
      }
    }
  }
}

#endif
```

- If we have too many previews, we can assign each a name (displayed below each preview) with the [`.previewDisplayName(“title”)` API`][previewNameDoc].

## Development assets

- New in Xcode 11
- Used (for example) when we want to preview images without having needing to relay on a network or with the need to add assets in our final binary.

## UIKit previews

- Works exactly like SwiftUI previews
- Create a `PreviewProvider`.
- Create a `UIViewRepresentable`/`UIViewControllerRepresentable`.
- Add `UIView`s/`UIViewController`s in the `PreviewProvider`'s `previews` static property.

## Canvas Protips:

- Use the pin button in the bottom left corner of the preview to pin the current preview and jump to other files.

- Show/hide the canvas by pressing `⌘` + `⌥` + `↩`.

## MVVM

Apple is really pushing us to use the [MVVM architecture][mvvmWiki] with SwiftUI: Model-view-viewmodel.

Apple even suggests to add tests on how data migrates from a model to a view model:
![][mvvmImage]

[previewProviderDoc]: https://developer.apple.com/documentation/swiftui/previewprovider
[previewDeviceDoc]: https://developer.apple.com/documentation/swiftui/securefield/3289399-previewdevice
[PreviewLayoutDoc]: https://developer.apple.com/documentation/swiftui/securefield/3289402-previewlayout
[forEachDoc]: https://developer.apple.com/documentation/swiftui/foreach
[previewNameDoc]: https://developer.apple.com/documentation/swiftui/pastebutton/3271204-previewdisplayname
[mvvmWiki]: https://en.wikipedia.org/wiki/Model–view–viewmodel

[mvvmImage]: WWDC19-233-mvvm
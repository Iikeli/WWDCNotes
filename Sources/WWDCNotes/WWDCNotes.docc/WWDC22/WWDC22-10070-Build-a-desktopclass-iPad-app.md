# Build a desktop-class iPad app

Discover how you can create iPad apps that take advantage of desktop class features. Join Mohammed from the UIKit team as we explore the latest navigation, collection view, menu, and editing APIs and learn best practices for building powerful iPad apps. Code along with this session in real time or download our sample app to use as a reference for updating your own code.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10070", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## `UINavigationItem` style

- [`UINavigationItem`][UINavigationItem] gains a new `style` instance property of type [`UINavigationItem.ItemStyle`][UINavigationItem.ItemStyle]

![][navStyleImg]

- `.navigator` - (default) behaves exactly as a traditional `UINavigationBar`
  - title is centered
  - there are leading and trailing bar button items
  - a back button appears when there is more than one item on the stack

- `.browser` - rearranges contents to be better optimized for interfaces where history matters as much as location (e.g., Files.app, Safari.app)
  - title is leading aligned

- `.editor` - optimized for when the primary function is document editing
  - title is leading aligned
  - editor UIs are often a destination, such as after selecting a document with a document picker, and so present a back button for easy access to that UI

Both `.browser` and `.editor` styles allow to add center items to take better advantage of screen real estate via the new [`centerItemGroups`][centerItemGroups] `UINavigationItem` instance property:

- include support for `UIBarButtonItemGroup`

Overflow support is available in all modes (a button that, when tapped, shows more buttons)

- allows the navigator style to indirectly support center items as well (as they will just be tucked away in this overflow button)

To allow navigation bar customization, set the [`customizationIdentifier`][customizationidentifier]

Example:

```swift
navigationItem.customizationIdentifier = "com.jetpack.blueprints.maineditor"
navigationItem.centerItemGroups = [
  // groups in the default customization
  UIBarButtonItem(title: "Insert", image: UIImage(systemName: "photo"), primaryAction: UIAction { _ in }).creatingFixedGroup(),
  UIBarButtonItem(title: "Draw", image: UIImage(systemName: "scribble"), primaryAction: UIAction { _ in }).creatingMovableGroup(customizationIdentifier: "Draw"),
  .optionalGroup(
    customizationIdentifier: "Shapes",
    representativeItem: UIBarButtonItem(title: "Shapes", image: UIImage(systemName: "square.on.circle")),
    items: [
        UIBarButtonItem(title: "Square", image: UIImage(systemName: "square"), primaryAction: UIAction { _ in }),
        UIBarButtonItem(title: "Circle", image: UIImage(systemName: "circle"), primaryAction: UIAction { _ in }),
        UIBarButtonItem(title: "Rectangle", image: UIImage(systemName: "rectangle"), primaryAction: UIAction { _ in }),
        UIBarButtonItem(title: "Diamond", image: UIImage(systemName: "diamond"), primaryAction: UIAction { _ in }),
      ]
  ),
  .optionalGroup(
    customizationIdentifier: "Text",
    items: [
      UIBarButtonItem(title: "Label", image: UIImage(systemName: "character.textbox"), primaryAction: UIAction { _ in }),
      UIBarButtonItem(title: "Text", image: UIImage(systemName: "text.bubble"), primaryAction: UIAction { _ in }),
    ]
  ),
  // additional group not in the default customization
  .optionalGroup(
    customizationIdentifier: "Format",
    isInDefaultCustomization: false,
    representativeItem: UIBarButtonItem(title: "BIU", image: UIImage(systemName: "bold.italic.underline")),
    items:[
      UIBarButtonItem(title: "Bold", image: UIImage(systemName: "bold"), primaryAction: UIAction { _ in }),
      UIBarButtonItem(title: "Italic", image: UIImage(systemName: "italic"), primaryAction: UIAction { _ in }),
      UIBarButtonItem(title: "Underline", image: UIImage(systemName: "underline"), primaryAction: UIAction { _ in }),
    ]
  )
]
```

## Document interaction

`UINavigationBar` now supports adding a menu to the title view

- add actions that operate on the content as a whole
- share sheet support
- drag & drop support

Default actions (these items are filtered based on specific methods in your responder chain):

- Duplicate
- Move
- Rename
- Export
- Print

To enable the title menu, set the [`titleMenuProvider`][titleMenuProvider], which is a closure that returns the final menu to be displayed:

```swift
navigationItem.titleMenuProvider = { suggestedActions in
  var children = suggestedActions
  children += [
    UIAction(title: "Comments", image: UIImage(systemName: "text.bubble")) { _ in }
  ]
  return UIMenu(children: children)
}
```

To enable drag and drop and sharing from the menu:

```swift
let url = ...
let documentProperties = UIDocumentProperties(url: url)

if let itemProvider = NSItemProvider(contentsOf: url) {
  documentProperties.dragItemsProvider = { _ in
    [UIDragItem(itemProvider: itemProvider)]
  }

  documentProperties.activityViewControllerProvider = {
    UIActivityViewController(activityItems: [itemProvider], applicationActivities: nil)
  }
}

navigationItem.documentProperties = documentProperties
```

## Inline Rename

- provided by setting [`UINavigationItem.renameDelegate`][UINavigationItem.renameDelegate]
- provides a dedicated UI for editing the title on all platforms

Example:

```swift
class ViewController: UIViewController {
  override func viewDidLoad() {
    super.viewDidLoad()
    navigationItem.renameDelegate = self
  }
}

extension ViewController: UINavigationItemRenameDelegate {
  func navigationItem(_ navigationItem: UINavigationItem, didEndRenamingWith title: String) {
    // Try renaming our document, the completion handler will have the updated URL or return an error.
    documentBrowserViewController.renameDocument(at: <#T##URL#>, proposedName: title, completionHandler: <#T##(URL?, Error?) -> Void#>)
  }
}
```

[navStyleImg]: navStyleImg.png
[UINavigationItem]: https://developer.apple.com/documentation/uikit/uinavigationitem
[style]: https://developer.apple.com/documentation/uikit/uinavigationitem/3987969-style
[UINavigationItem.ItemStyle]: https://developer.apple.com/documentation/uikit/uinavigationitem/itemstyle
[centerItemGroups]: https://developer.apple.com/documentation/uikit/uinavigationitem/3987967-centeritemgroups
[customizationidentifier]: https://developer.apple.com/documentation/uikit/uinavigationitem/3987968-customizationidentifier
[titleMenuProvider]: https://developer.apple.com/documentation/uikit/uinavigationitem/3967523-titlemenuprovider
[UINavigationItem.renameDelegate]: https://developer.apple.com/documentation/uikit/uinavigationitem/3967522-renamedelegate
# Build document-based apps in SwiftUI

Learn how to build a document-based app entirely in SwiftUI! We’ll walk you through the DocumentGroup API and how it composes with your App and Scenes, allowing you to add out-of-the-box support for document management — such as document browsing and standard commands — no heavy lifting required. You’ll learn to set up Universal Type Identifiers as well as gain understanding into what makes a top-notch document-based app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10039", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



Document-based apps are apps whose primary workflow is opening user's files of a specific type from the disk, editing them and saving them back to disk - as opposed to apps that work with data kept in their own internal databases.

There are also apps of mixed type, where document-focused features are a part of the UI, e.g.:

- Xcode is primarily document-based, but it also has non-document parts like documentation, organizer
- Mail generally works with an internally managed database, but can also open single email files from any place on disk

For document-based apps, use the [`DocumentGroup`](https://developer.apple.com/documentation/swiftui/documentgroup) scene as the main scene of the app:

```swift
DocumentGroup(newDocument: MyDocument()) { file in
    TextEditor(text: file.$document.text)
}
```

You can also mix `DocumentGroup` and `WindowGroup` in apps that should have both document-based and non-document-based windows.

`DocumentGroup` automatically provides various document management features expected on the given platform:

- menu items like File > Open/Save in the macOS menu
- state tracking and file renaming/moving using the titlebar popup on macOS
- Handoff between devices
- a document browser and a navigation bar with search field on iOS

The `DocumentGroup` initializer takes a document object which represents a single document of a type handled by the given document group, and a view closure that describes the view hierarchy to be rendered for each document's scene. The closure's argument (`file`) gives you a binding that provides read-write data to the document's data model. The binding lets SwiftUI know when you update the document contents through the binding, so it can automatically handle things like registering undo actions and marking the document state as dirty.

[`TextEditor()`](https://developer.apple.com/documentation/swiftui/texteditor) - a new built-in text view type kind of like `UITextView`


## Document types

Document-based apps have a “Document Types” section in the `Info.plist`, where you declare Uniform Type Identifiers of document types associated with your app:

- Imported types ⭢ documents from other apps or sources that your app is able to open
- Exported types ⭢ document types owned by your app


The type that implements the document model conforms to [`FileDocument`](https://developer.apple.com/documentation/swiftui/filedocument) (for value types) and declares its own [`UTType`](https://developer.apple.com/documentation/uniformtypeidentifiers/uttype) instances that represent imported and exported file types:

```swift
import UniformTypeIdentifiers

extension UTType {
    static var exampleText: UTType {
        UTType(importedAs: "com.example.plain-text")
    }

    static let shapeEditDocument =
        UTType(exportedAs: "com.example.ShapeEdit.shapes")
}
```

An imported type needs to be a computed property, because the value returned from the constructor may change between calls while the app is running, depending on system configuration. Exported type can just be assigned once and stored. (This is also explained in a tech talk video which talks specifically about UTIs, "[Uniform Type Identifiers — a reintroduction](https://developer.apple.com/videos/play/tech-talks/10696/)".)

The document type provides a list of types (public types and app's own types) that it can accept:

```swift
static var readableContentTypes: [UTType] { [.exampleText] }
```

It also has methods for reading and writing its document to/from a file, which you need to implement, using e.g. Codable to encode/decode the value into your chosen format:

```swift
init(fileWrapper: FileWrapper, contentType: UTType) throws { ... }

func write(to fileWrapper: inout FileWrapper, contentType: UTType) throws { ... }
```

In those methods, you can assume that the content type is one of those you declared as accepted by your app.

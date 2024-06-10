# Meet Transferable

Meet Transferable: a model-layer protocol that allows for effortless support for sharing, drag and drop, copy/paste, and other features in your app.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10062", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- sharing is now supported on watchOS 9

## Content type (Uniform type identifier - UTI)

- Apple-specific technology that describes identifiers for different binary structures as well as abstract concepts
- in order to declare a custom identifier: 
  1. add its declaration to your app <kbd>Info.plist</kbd> file
  2. (recommended) create a file extension: if the data is saved on disk, the system will know that your app can open that file
  3. declare it in code

```swift
import UniformTypeIdentifiers

// also declare the content type in the Info.plist
extension UTType {
  static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

## `Transferable`

- Swift-first declarative way to describe how your models can be serialized and deserialized for sharing and data transfer
- supports standard `Transferable` types: `String`, `Data`, `URL`, Attributed String, `Image`

### Use example

- new [`PasteButton`][PasteButton]:

```swift
var body: some View {
  // 👇🏻 allow pasting a Transferable type
  PasteButton(payloadType: String.self) { pastedString in
    // ...
  }
}
```

- new [`draggable(_:)`][draggable(_:)] and [`dropDestination(payloadType:action:isTargeted:)`][dropDestination(payloadType:action:isTargeted:)] modifiers:

```swift
struct PortraitView: View {
  @State var portrait: Image // 👈🏻 Transferable type

  var body: some View {
    portrait
      .cornerRadius(8)
      .draggable(portrait) // 👈🏻 drag support
      .dropDestination(payloadType: Image.self) { (images: [Image], _) in // 👈🏻 drop support
        if let image = images.first {
          portrait = image
          return true
        }
        return false
      }
  }
}
```

- new [`ShareLink`][ShareLink]:

```swift
var body: some View {
  VStack {
    portrait
    Text(model.name)
  }
  .toolbar {
    ShareLink(item: portrait, preview: SharePreview(model.name))
  }
}
```

## Conforming custom types to `Transferable`

- `Transferable` definition:

```swift
public protocol Transferable {
  associatedtype Representation: TransferRepresentation

  @TransferRepresentationBuilder<Self> static var transferRepresentation: Self.Representation { get }
}

public protocol TransferRepresentation: Sendable {
  associatedtype Item: Transferable
  associatedtype Body: TransferRepresentation

  @TransferRepresentationBuilder<Self.Item> var body: Self.Body { get }
}
```

- to conform a type to `Transferable`, there's only one property to implement: `transferRepresentation` (of type `TransferRepresentation`)

- There are three important representations to be aware of:
 - `CodableRepresentation`
 - `DataRepresentation`
 - `FileRepresentation`

- if our type conforms to `Codable`, we can use the `CodableRepresentation`
  - by default `Transferable` uses a `JSONEncoder`/`JSONDecoder` , but you can provide your own

```swift
import CoreTransferable
import UniformTypeIdentifiers

struct Profile: Codable, Transferable { // 👈🏻
  var id: UUID
  var name: String
  var bio: String
  var funFacts: [String]
  var video: URL?
  var portrait: URL?

  // MARK: Transferable

  static var transferRepresentation: some TransferRepresentation { // 👈🏻
    CodableRepresentation(contentType: .profile)
  }
}

// also declare the content type in the Info.plist
extension UTType {
  static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

- `DataRepresentation` vs. `FileRepresentation`
  - choose the former if the model is always in-memory
  - choose the latter if the model can be stored on disk
  - `FileRepresentation` is intended to work with the URLs written to disk, and ensure receivers' access to this file or its copy by granting a temporary sandbox extension

- we can pass multiple representations in the `transferRepresentation` property
  - the receiver will use the first representation with the content type they support.

```swift
import CoreTransferable
import UniformTypeIdentifiers

struct Profile: Codable {
  var id: UUID
  var name: String
  var bio: String
  var funFacts: [String]
  var video: URL?
  var portrait: URL?
}

extension Profile: Transferable {
  static var transferRepresentation: some TransferRepresentation {
  CodableRepresentation(contentType: .profile) // 👈🏻 first representation
  ProxyRepresentation(exporting: \.name) // 👈🏻 alternative representation
  }
}

// also declare the content type in the Info.plist
extension UTType {
  static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

[dropDestination(payloadType:action:isTargeted:)]: https://developer.apple.com/documentation/swiftui/view/dropdestination(payloadtype:action:istargeted:)
[draggable(_:)]: https://developer.apple.com/documentation/swiftui/view/draggable(_:)
[PasteButton]: https://developer.apple.com/documentation/swiftui/pastebutton
[ShareLink]: https://developer.apple.com/documentation/swiftui/sharelink
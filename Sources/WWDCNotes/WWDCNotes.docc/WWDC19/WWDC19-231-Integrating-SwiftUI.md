# Integrating SwiftUI

SwiftUI is designed to integrate with your existing code base on any of Apple’s platforms. Learn how to adopt SwiftUI on any Apple platform by adding SwiftUI views into your app’s hierarchy, leveraging your existing data model and more.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/231", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



We can host SwiftUI content in your UIKit views via a [`UIHostingController`][HostDoc].

## Representable Protocol

- Lets us embed `UIView`s in SwiftUI views (via [`UIViewRepresentable`][viewRepDoc])
- Lets us embed child `ViewController`s (via [`UIViewControllerRepresentable`][vcRepDoc])

The protocol has three methods:

- `Make`: where we create the view or view controller that we want to present in SwiftUI.
- `Update`: where we update this view/ or view controller to the current configuration.
- `Dismantle`: optional.

During initialization, the `make` method is called first, followed by the `update` method.

The `update` method can be called multiple times, whenever an update is requested by SwiftUI.

## Representable Context

Used for advanced integration: perhaps we want to expose target action or delegation in SwiftUI, or we may want to read from the environment of SwiftUI and respond accordingly.

### [`Coordinator`][cooDoc]

- Helps with coordinating between UIVIew/Controller and SwiftUI
- Can be used to implement common patterns, like delegation, data sources, and target action.

### `Environment`

- Helps read SwiftUI environment 

### `Transaction`

- Lets our view know if there was an animation in SwiftUI.

## Item Providers

- Item providers are a great technology that's provided by `Foundation`, which provides us a means of moving data around your application in various forms.
- It's also a tool that we use to help transfer data across processes.
- We can use them to integrate with system services like drag and drop (by using [`.onDrag`][dragDoc] and [`.onDrop`][dropDoc]), pasteboard (via [`.onPasteCommand`][pasteDoc])

[HostDoc]: https://developer.apple.com/documentation/swiftui/uihostingcontroller
[viewRepDoc]: https://developer.apple.com/documentation/swiftui/uiviewrepresentable
[vcRepDoc]: https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable
[cooDoc]: https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable/3309742-makecoordinator
[dragDoc]: https://developer.apple.com/documentation/swiftui/view/3289004-ondrag
[dropDoc]: https://developer.apple.com/documentation/swiftui/view/3289005-ondrop
[pasteDoc]: https://developer.apple.com/documentation/swiftui/text/3367731-onpastecommand
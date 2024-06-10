# Meet Scribble for iPad

Scribble offers a lightweight, ergonomic, and enjoyable way of entering text on iPad with Apple Pencil. Discover how people can take advantage of Scribble and handwritten text in apps that use standard text input controls or that implement a custom text editing experience. You’ll learn how it integrates into TextKit, and when you’ll need to adopt the new UIScribbleInteraction and UIIndirectScribbleInteraction APIs to provide a delightful and consistent experience with Scribble in your app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10106", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Writing experience

- You can write directly on a text field without the need to select the field first.
- You can draw a horizontal line to select text and perform the normal text editing actions.
- You can also scratch out some text that you want to delete.

## Scribble Supporting APIs

- Scribble relies on standard Text controls/inputs (`UITextField`, `UITextView`, ...)
- Scribble does not support password fields (because Apple's recommended way to input password is via auto fill)
- for custom UI elements Scribble relies on UIKit's text input protocols ([`UITextInput`][UITextInput], [`UIKeyInput`][UIKeyInput], [`UITextInputTraits`][UITextInputTraits], [`UITextInteraction`][UITextInteraction])

## New APIs

Scribble introduces two interactions, [`UIScribbleInteraction`][UIScribbleInteraction] and [`UIIndirectScribbleInteraction`][UIIndirectScribbleInteraction], which are used to customize the behavior of Scribble in your app.

Both interactions are added to views where the custom behavior takes place.

### UIScribbleInteraction

The interaction has a delegate, and this delegate is where an app can customize the Scribble experience, for example:

- disabling Scribble on the view
- delaying that view from becoming first responder until handwriting has momentarily paused
- being informed when Scribble handwriting begins or ends

### UIIndirectScribbleInteraction

Thanks to this interaction:

- we can allow writing outside a input control.
- this is also the interaction to use for UI that would become editable in response to a tap gesture.

This is done via the interaction's delegate. Which will provide the system information about elements, aka regions within the view that can be written into.

[UIIndirectScribbleInteraction]: https://developer.apple.com/documentation/uikit/uiindirectscribbleinteraction
[UITextInput]: https://developer.apple.com/documentation/uikit/uitextinput
[UIKeyInput]: https://developer.apple.com/documentation/uikit/uikeyinput
[UITextInputTraits]: https://developer.apple.com/documentation/uikit/uitextinputtraits
[UITextInteraction]: https://developer.apple.com/documentation/uikit/uitextinteraction
[UIScribbleInteraction]: https://developer.apple.com/documentation/uikit/uiscribbleinteraction
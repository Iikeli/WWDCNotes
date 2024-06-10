# Build Mail app extensions

Meet MailKit: the best way to build amazing experiences on top of Mail. MailKit enables apps to easily and securely interact with the Mail app for macOS. We'll deep dive into the MailKit API, and show you how to create extensions for composing messages, message actions, secure email, and content blocking.


@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10168", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample code](https://developer.apple.com/documentation/mailkit/build_mail_app_extensions)

- Mail plug-ins will stop working in a future macOS release
- New [MailKit framework][MailKit], macOS 12.0+
- Mail extensions are built on the same underlying foundation as other app extensions, like Safari app extensions and share sheet extensions
- can be distributed standalone, bundled in an app (even notarized ones)

Extension types:

- compose extensions - allow new workflows when composing mail messages
- action extensions - help people manage their inbox by providing custom rules on incoming messages
- content blocking extensions - provide WebKit content blockers for Mail messages
- message security extensions - provide further security by signing, encrypting, and decrypting messages when people send and receive mail

## Creating a new Mail Extension

When you create a new Mail extension, you will be asked which types (among the ones above) your extension will be. This will enable/disable capabilities on your extension accordingly.

The principal class of your extension must conform to the [`MEExtension`][MEExtension] protocol. 

- `MEExtension` exposes optional handler methods for each of the four types of extensions
- All the methods in `MEComposeSessionHandler` have a `MEComposeSession` argument which provides information about a compose window. Mail creates a unique `MEComposeSession` instance for every Mail compose window. Each window has a `MEMessage` property that exposes various details of the message being edited.

## Compose extensions

Four ways your extension can interact with a Mail compose window:

- validate/annotate recipient email addresses; as the user is editing them
- present a view controller with additional context about the message being composed; the view controller must be a subclass of [`MEExtensionViewController`][MEExtensionViewController]
- set additional headers on outgoing messages
- validate/alert the user of errors in the message before it is sent

For `MEComposeSession` you will need to provide a `MEComposeIcon` and `MEComposeIconToolTip` entries in your extension <kbd>Info.plist</kbd>:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>NSExtension</key>
  <dict>
    <key>NSExtensionAttributes</key>
    <dict>
      <key>MEComposeSession</key>
      <dict>
        <key>MEComposeIcon</key> // 👈🏻
        <string>lock</string>
        <key>MEComposeIconToolTip</key> // 👈🏻
        <string>Recipient Validation Extension</string>
      </dict>
    </dict>
    <key>NSExtensionPointIdentifier</key>
    <string>com.apple.email.extension</string>
    <key>NSExtensionPrincipalClass</key>
    <string>$(PRODUCT_MODULE_NAME).MailExtension</string>
  </dict>
</dict>
</plist>
```

Example on how to validate recipient addresses:

```swift
func annotateAddressesForSession(_ session: MEMailComposeSession) async -> [String: MEAddressAnnotation] {
  var annotations: [String: MEAddressAnnotation] = [:]
  
  // Iterate through all the recipients in the message.
  for address in session.mailMessage.allRecipientAddresses {
    // Annotate invalid recipients with an error.
    if address != "seth@example.com" {
      let message = "example.com is not a valid domain"
      let annotation = MEAddressAnnotation.error(withLocalizedDescription: message)
      annotations[address] = annotation
    }
  }

  return annotations
}
```

## Action extensions

Three types:

- mark messages as read and/or flag messages
- move to other standard system mailboxes (e.g., Junk, Trash or Archive)
- apply colors on messages in the message list

- Your action extension must implement [`MEMessageActionHandler`][memessageactionhandler]'s [`decideAction(for:completionHandler:)`][decideAction(for:completionHandler:)]
- `decideAction(for:completionHandler:)` is called with a `MEMessage` argument
- Mail calls your handler's decideAction for message for every new message that it downloads before it is even visible in the inbox.

## Content blocking extensions

- Your content blocking extension must implement an object conforming to [`MEContentBlocker`][MEContentBlocker]
- The content rule lists are specified using the same syntax as [Safari content blockers][scb], e.g.:

```xml
[
  {
    "action": { "type": "block" },
    "trigger": { "url-filter": "*.acme.*" }
  },
  ...
]
```

## Message security extensions 

- Your content blocking extension must implement an object conforming to [`MEMessageSecurityHandler`][MEMessageSecurityHandler]
- each time the sender or recipients change, Mail will call the `getEncodingStatus(for:completionHandler:)` method on the extension's message security handler
- when the message is sent, Mail will take the <kbd>RFC822</kbd> message data and pass it to the extension
  - the extension will sign and encrypt the message as needed, and return the signed and encrypted <kbd>RFC822</kbd> data back to Mail. Mail will then send this data to the outgoing server

[MEMessageSecurityHandler]: https://developer.apple.com/documentation/mailkit/memessagesecurityhandler
[scb]: https://developer.apple.com/documentation/safariservices/creating_a_content_blocker
[MEContentBlocker]: https://developer.apple.com/documentation/mailkit/mecontentblocker
[decideAction(for:completionHandler:)]: https://developer.apple.com/documentation/mailkit/memessageactionhandler/3783568-decideaction
[memessageactionhandler]: https://developer.apple.com/documentation/mailkit/memessageactionhandler
[MEExtensionViewController]: https://developer.apple.com/documentation/mailkit/meextensionviewcontroller
[MailKit]: https://developer.apple.com/documentation/mailkit
[MEExtension]: https://developer.apple.com/documentation/mailkit/meextension
# Verify app dependencies with digital signatures

Discover how you can help secure your app’s dependencies. We’ll show you how Xcode can automatically verify any signed XCFrameworks you include within a project. Learn how code signatures work, the benefits they provide to help protect your software supply chain, and how SDK developers can sign their XCFrameworks to help keep your apps secure.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10061", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(MarcoEidinger)
   }
}



Signature verification of binary frameworks is a new feature in Xcode 15 that provides security for **app developers** based on code signing done by **SDK authors**. 

## Must-know for **App Developers**

Xcode 15 automatically verifies your app's dependencies and protects your supply chain integrity by alerting you if an issue is detected.

Xcode now shows a new section in the Inspector that displays the signature status for the xcframeworks you include within your app.

![Signature status][xcodesignaturestatus]

[xcodesignaturestatus]: WWDC23-10061-xcodesignaturestatus


This section includes information about the signature, such as the author's identity. It will show whether the xcframework is signed by

- an Apple Developer Program identity
- a self-signed certificate or
- is currently unsigned

Xcode will record the identity the first time you use the xcframework and verify that it does not change during later builds.

![Signature changed][signaturechanged]

[signaturechanged]: WWDC23-10061-signaturechanged

If the signature changed your app's build will fail with an appropriate error message.

![Build failed on signature change][signatureverificationfailed]

[signatureverificationfailed]: WWDC23-10061-signatureverificationfailed

Selecting the error offers an alert explaining that the developer's identity has changed, showing how the expected identity compares to what is present in the new version.

The change may indicate that the binary framework was compromised. The change could be legitimate. Contact the SDK author for clarification through a verified public channel if necessary.

Cancel if you are not sure. This will move the new version to trash in Xcode. 

![Confirm or discard change][confirmordiscardsignaturechange]

[confirmordiscardsignaturechange]: WWDC23-10061-confirmordiscardsignaturechange

[Article: Verifying the origin of your XCFrameworks](https://developer.apple.com/documentation/Xcode/verifying-the-origin-of-your-xcframeworks)

## Must-know for **SDK Authors**

SDK authors are highly encouraged to sign their XCFrameworks but are not forced to.

> It is important for SDK authors to cryptographically sign SDKs because it allows an app developer to confirm the identity and guarantees that the code has not been altered or tampered with after it was signed.

SDK authors are encouraged to use their Apple Developer Program identity for signing. For different developer identities, Xcode provides differing levels of features based on the trust in the identity that was used signing.

![Signing XCFrameworks - Identity types][identitytypes]

[identitytypes]: WWDC23-10061-identitytypes

Use the `codesign` tool to sign your binary frameworks. The example below is using an Apple Developer Program identity.

```
codesign --timestamp -v --sign "Apple Distribution: Truck to Table (UA527FUGW7)" BirdFeeder.xcframework
```

The timestamp flag is included to ensure that the signature contains a secure timestamp that is attested to by Apple.

The signature resides within the `_CodeSignature` directory of your XCFramework and protects the integrity of all the files inside your final binary framework.

SDK authors are responsible for sharing with their SDK clients the fingerprint of this certificate so that app developers can verify the xcframework was signed by the SDK author. 
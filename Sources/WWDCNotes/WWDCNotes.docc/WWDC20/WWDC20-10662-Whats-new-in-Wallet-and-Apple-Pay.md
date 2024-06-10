# What's new in Wallet and Apple Pay

Apple Pay makes it simple to pay for goods and services in your app and on your website. Discover how you can integrate API updates like context-specific button types, contact data formatting, and cross-platform support to make the service more effective for you and people using it. And, if you’re building app clips, adopting Apple Pay can help you unlock new commerce experiences.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10662", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



* [`PKPaymentPass`][paymentPassDoc] has been replaced by a new class [`PKSecureElementPass`][secureElementDoc]. 
* `PKSecureElementPass` has the same properties as `PKPaymentPass`, so it’s encouraged to use this one.

* [`PKPaymentButton`][paymentButtonDoc] didn’t change much, however it has new [button types][buttonTypes] for renting, top up, etc..
* Now the payment button can derive its light/dark style theme from the system by supplying an automatic style to the button:

```swift
var automaticButton = PKPaymentButton(paymentButtonType: .plain, paymentButtonStyle: .automatic)
```

> We can also request specifically for light or dark mode in the same way

* Now Catalyst and native macOS apps supports paying with Apple Pay in the same way it’s implemented on web pages
* Also, instead of asking the WebKit for a URL for Apple Pay and validate that URL against known Apple hostnames, there is a static URL you we always send our `POST` request to directly: `apple-pay-gateway.apple.com`

## Contacts formatting improvements

* in iOS 14, when contact data is coming from Apple Pay, qw will see more consistent data and less variation
* Also, users are now prompted to correct issues in their contact information earlier, so they get to fix it sooner in the process.

[paymentPassDoc]: https://developer.apple.com/documentation/passkit/pkpaymentpass
[secureElementDoc]: https://developer.apple.com/documentation/passkit/PKSecureElementPass
[paymentButtonDoc]: https://developer.apple.com/documentation/passkit/PKPaymentButton
[buttonTypes]: https://developer.apple.com/documentation/passkit/pkpaymentbuttontype

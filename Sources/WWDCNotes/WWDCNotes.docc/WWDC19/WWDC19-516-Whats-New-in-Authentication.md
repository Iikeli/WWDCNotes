# What's New in Authentication

Secure sign-in and authentication is a key feature of a secure account-based app design. Learn how you can improve your app's login experiences through an overview of the available authentications services and details on specific technologies such as Sign In with Apple ID, Password AutoFill for iPad Apps for Mac, advances in OAuth and WebAuthentication, and a new API for streamlined password sign-in.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/516", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(MTSchmidt)
   }
}



## Sign in with Apple

- Private alternative to 3rd party logins
- Authentication via Face ID / Touch ID (2-factor built-in)
- Verifying email addresses is already done by Apple (Apple ID)
- Claimed to be easier than passwords
- Across platforms
  - iOS 13+
  - iPadOS 13+
  - watchOS 6+
  - macOS 10.15+
  - tvOS 13+
  - Safari 13+
  - Other platforms via JavaScript (Windows, Android and other browsers)

- On app launch, app can check for existing password-based user accounts via Keychain

## Password-based Authentication

- Password Auto-fill available for iPad apps on the Mac
  - New app ID has to be listed on the server via apple-app-site-association file
  - New keys for Universal Links

- Authentication services can be used for apps (check associated domains to combine app and website)
- Can request multiple types of accounts via [`ASAuthorizationController`][ASAuthorizationController] (e.g. password-based + Sign in with Apple)

## Warnings for weak passwords

- iPhone + Mac have Automatic Strong Passwords
- Safari 13 on iOS 13 shows weak passwords + possibility to change it
  - Forwarding to website where user can change password
  - Redirect to https://example.com/.well-known/change-password (client-side or server-side)
  - [`Well-Known URL for Changing Passwords`][well-known-url]
  
## OAuth sign-in
  
- Available on iOS, and now macOS
  - macOS uses preferred web browser
  - If available, browser's password manager or password manager extension will help

- OAuth with ASWebAuthenticationSession
  - More private sign-in and better UX
    - No alert anymore but redirect to identify provider native UI
    - Set [`prefersEphemeralWebBrowserSession`][prefersEphemeralWebBrowserSession] = `true`

- 
  - Supporting multiple windows due to iPadOS and macOS support
  - Migrate from `SFAuthenticationSession` to [`ASWebAuthenticationSession`][ASWebAuthenticationSession]
  
## USB security keys on macOS

- Safari 13 supports USB-based FIDO2-compliant devices with the WebAuthentication standard
- Offer great account recovery story, like using previous mentioned services

[ASAuthorizationController]: https://developer.apple.com/documentation/authenticationservices/asauthorizationcontroller
[well-known-url]: https://wicg.github.io/change-password-url/
[prefersEphemeralWebBrowserSession]: https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/3237231-prefersephemeralwebbrowsersessio
[ASWebAuthenticationSession]: https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession

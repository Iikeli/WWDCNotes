# Introducing Sign In with Apple

Sign In with Apple is the fast, easy way for people to sign in to apps using the Apple IDs they already have. Learn how easy it is to add a Sign In with Apple button to your app or website to acquire new customers and benefit from the built-in security, antifraud, and privacy that Sign In with Apple provides.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/706", purpose: link, label: "Watch Video (35 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- **Secure** - backed by 2FA of the AppleID
- **Private** - not tracked by Apple
- **Fast and Easy** - user doesn't even need a keyboard
- **User controls** which data to share
- **Seamless Across Device** recognizes that already signed in on other device
- **Requires Capability** for Apple Sign In has to be added
- **Private E-Mail Relay** links random mail to your AppleID
  - Apple never retains messages
  - Can be used for any email communication like receipts, ...
  - Two-Way Relay

- **Anti-Fraud Detection** can tell if a robot tries to sign in or not
- **Cross Platform** iOS, macOS, watchOS, tvOS, JavaScript (Android Websites)
- **ASAuthorizationAppleIDButton** creates button that has different visual styles and labels
- **Authentication Request** returns 
  - **UserID** which is unique, stable and team-scoped and can be used as the key to the user
  - **Verification Data** identity token and short-lived code to refresh token
  - **Full Name** as PersonNameComponents which contain first/last name separately
  - **Verified email** your server doesn't need to verify this email again
  - **Real User Indicator** high confidence indicator that likely real user
  - **Credential State** tells if UserID is `authorized` (let user pass), `revoked` (handle unlink) or `not Found` (show login)

- **Always check on AppStart** with `provider.getCredentialState(userID)` which runs very fast
- Listen to `NSNotification.Name.ASAuthorizationAppleIDProviderCredentialRevoked` and sign the user out if called
- **Password Autofill** integrates with Apple Sign In. Triggered if the device detects a stored credential

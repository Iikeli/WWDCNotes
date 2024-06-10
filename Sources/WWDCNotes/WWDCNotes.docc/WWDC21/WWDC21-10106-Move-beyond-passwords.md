# Move beyond passwords

Despite their prevalence, passwords inherently come with challenges that make them poorly suited to securing someone’s online accounts. Learn more about the challenges passwords pose to modern security and how to move beyond them. Explore the next frontier in account security with secure-by-design, public-key-based credentials that use the Web Authentication standard. Discover in this technology preview how Apple is approaching this standard in iOS 15 and macOS Monterey.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10106", purpose: link, label: "Watch Video (25 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample code](https://developer.apple.com/documentation/authenticationservices/connecting_to_a_service_with_passkeys)

## Authentication methods

- Memorized passwords; people generally aren’t good at coming up with and remembering strong and unique passwords for every account
- Password managers; can create strong, unique passwords per account, and can provide hints about some forms of possible phishing. iCloud Keychain’s password manager is built into iOS/macOS
- Password + OTP; macOS Monterey and iOS 15 have a code generator built in to the iCloud Keychain password manager
- Federated authentication; lets people keep their trust confined to a small number of highly protected accounts (e.g., Sign in with Apple)

Besides federated authentication, all of these methods rely on a shared secret between the user and a server, making them fundamentally no stronger than the weakest protection of that shared secret.

## Properties of a password solution:

- secure by design
- easy to use
- works everywhere
- recoverable

## Security keys (hardware dongles or fobs)

- One of the strongest security options available today
- [WebAuthn standard](https://en.wikipedia.org/wiki/WebAuthn) (essentially RSA)
- Generally easy to use
- More secure than passwords
- usable on a wide variety of devices
- no server-side secrets
- trust comes from the browser and operating system; not the human
- credentials are only ever usable on the websites and apps that they were created for

Cons:

- they’re not necessarily always with you
- if the key is lost/stolen, adopters must have a backup system in place, such as purchasing an additional security key, storing it somewhere safe, and hoping they never lose both at the same time

Security keys support:

- all browsers on iOS 14.5+
- new security key API available from macOS Monterey and iOS 15, available to use in all apps on macOS and iOS, part of [`ASAuthorization`][ASAuthorization]

## Passkeys in iCloud Keychain

- Technology preview (off by default, only use for testing)
- Apple’s contribution to a post-password world
- builds the security of WebAuthn into every iPhone, iPad, and Mac
- powered by WebAuthn
- new type of credential, called a <kbd>passkey</kbd>, stored in your iCloud Keychain

Comparison

|   | Memorized passwords | Password manager | Password + OTP | Security key | Passkeys in iCloud Keychain |
| --- | --- | --- | --- | --- | --- |
| Easy to use | ✅ | ✅ | ✅ | ✅ | ✅ | 
| Works on all your Apple devices | ✅ | ✅ | ✅ | ✅ | ✅ |
| Works on non-Apple devices | ✅ | ✅ | ✅ | ⚠️ | ⚠️ |
| Always with you | ✅ | ✅ | ✅ | ❌ | ✅ |
| Security level | ❌ | ⚠️ | ⚠️ | ✅ | ✅ |
| Recoverable | ❌ | ⚠️ | ⚠️ | ❌ | ✅ |
| Phishing resistant | ❌ | ⚠️ | ⚠️ | ✅ | ✅ |
| Doesn't require shared secrets | ❌ | ❌ | ❌ | ✅ | ✅ |

## How to support passkeys in iCloud Keychain

- for that strong, platform-provided phishing protection to work for your apps, the device needs a strong association between your app and website. This is done through associated domains, using the <kbd>webcredentials</kbd> association type.

E.g., this is the file returned from your website at <kbd>https://example.com/.well-known/apple-app-site-association</kbd>:

```json
{
  "webcredentials": {
    "apps": [ "A1B2C3D4E5.com.example.Shiny" ]
  }
}
```

How to register an account:

- the function needs three inputs:
  1. a single-use challenge fetched from your server
  2. the username for the account
  3. the userID; generally the identifier for the account on your backend

```swift
func createAccount(with challenge: Data, name: String, userID: Data) {
  let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(relyingPartyIdentifier: "example.com")

  let registrationRequest = provider.createCredentialRegistrationRequest(challenge: challenge, name: name, userID: userID)

  let controller = ASAuthorizationController(authorizationRequests: [registrationRequest])

  controller.delegate = …
  controller.presentationContextProvider = …

  controller.performRequests()
  // the perform request will trigger an iOS/macOS sheet to popup to prompt the 
  // user to create a new secret.
  // When the transaction is finished, you’ll receive a delegate callback with 
  // the details of the new credential.
}
```

Sign in:

```swift
func signIn(with challenge: Data) {
  let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(relyingPartyIdentifier: "example.com")

  let assertionRequest = provider.createCredentialAssertionRequest(challenge: challenge)


  let controller = ASAuthorizationController(
      authorizationRequests: [ assertionRequest ])

  controller.delegate = …
  controller.presentationContextProvider = …

  controller.performRequests()
}
```

Handle returned credentials, this is the delegate function called whenever the authorization completes:

```swift
func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
  switch authorization.credential {
    case let registration as ASAuthorizationPlatformPublicKeyCredentialRegistration:
      let attestationObject = registration.rawAttestationObject
      let clientDataJSON = registration.rawClientDataJSON
      // Verify on your server and finish creating the account.

    case let assertion as ASAuthorizationPlatformPublicKeyCredentialAssertion:
      let signature = assertion.signature
      let clientDataJSON = assertion.rawClientDataJSON
      // Verify on your server and finish signing in.

    case …:
      …
  }
}
```

[ASAuthorization]: https://developer.apple.com/documentation/authenticationservices/asauthorization
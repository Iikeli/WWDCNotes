# Meet passkeys

It’s time for a security upgrade: Learn how to add support for passkeys to create a quick and easy sign in experience for people, all while offering a radical increase to account security. Passkeys are simple and strong credentials built to eliminate phishing attacks. We’ll share how passkeys are designed with security in mind, show you how people will use them, go over how to integrate passkeys in your log in flow, and explore the platform and web APIs you need to adopt this feature.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10092", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## Introduction

- _next generation authentication technology_
- Passwords
  - Difficult to use correctly
  - Security vs convenience tradeoffs
  - Phishable and reusable

- Currently 2-factor authentication takes several steps:
  - add passkey button after being signed in
  - system sheet for creating a passkey
  - device generates a passkey

- With passkey: just select the user name, Face ID, and you’re signed in!
- Passkeys works on the web, too
- With FIDO alliance the passkeys work cross-platform
- When on a friends computer, just scan QR code with the phone, it recognizes that it’s for sign in with passkey
- Looks simple, but does lots of steps in the background, including proximity check and agreements
- Sharing passkey possible, e.g. via AirDrop

## Designing for passkeys

- Passkeys are replacements for passwords
- Refer to them with the noun “passkey”, pluralizes to “passkeys”
- Don’t design new interfaces for logins, just remove the password text field, but keep the user name field

## Passkeys and AutoFill

- Built on WebAuthn standard, use public key cryptopgraphy
- Require WebAuthn backend adoption
- Part of ASAuthorization API
- Supports many different credential types, new flexible UI options
- Setup associated domains first:

```json
{
  "webcredentials": {
    "apps": [ "A1B2C3D4E5.com.example.Shiny" ]
  }
}
```

- Make sure to use the `.username` text content type on text field
- Sign in logic via creating a request and starting the controller

```swift
func signIn() {
  let challenge: Data = … // Fetched from server
  let provider =
    ASAuthorizationPlatformPublicKeyCredentialProvider(
      relyingPartyIdentifier: "example.com")
  let request =
    provider.createCredentialAssertionRequest(
      challenge: challenge)

  let controller =
    ASAuthorizationController(
      authorizationRequests: [request])
  controller.delegate = self
  controller.presentationContextProvider = self

  // Start the request
  controller.performAutoFillAssistedRequests()
}
```

- System will offer autofill when this logic has been run before the text field is clicked by user
- Nothing gets filled in text field, instead this callback will trigger:

```swift
func authorizationController(controller: ASAuthorizationController,
   didCompleteWithAuthorization authorization: ASAuthorization) {
  
  guard let passkeyAssertion = authorization.credential as?
    ASAuthorizationPlatformPublicKeyCredentialAssertion
  else { … }

  let signature = passkeyAssertion.signature
  let clientDataJSON = passkeyAssertion.rawClientDataJSON

  // Pass these values to your server, and complete the sign in
  …
}
```

- make sure to verify sign in on the backend using the signature & data
- Same code also allows passkey sign in from nearby devices
- Apple was not showing the password field, only when no passkey (maybe a new best practice?)
- there’s also a modal request, just call `performRequests` instead, it will present a modal sheet with all available keys
- On web, annotate your username field with `WebAuthn`

```html
<input type="text" id="username-field" autocomplete="username webauthn" > 
```

- AutoFill-assisted WebAuthn request (JavaScript)

```js
function signIn() {
  if (!PublicKeyCredential.isConditionalMediationAvailable || !PublicKeyCredential.isConditionalMediationAvailable()) {
    // Browser doesn't support AutoFill-assisted requests.
    return;
  }

  const options = {
    "publicKey": {
      challenge: … // Fetched from server
    },
    mediation: "conditional"
  };

  navigator.credentials.get(options)
    .then(assertion => { 
      // Pass the assertion to your server.
    });
}
```

- Switching to a modal request by removing the `mediation` parameter
- Streamlining sign-in
- Modal request with allow lists:

```js
func signIn(userName: String) {
  let challenge: Data = … // Fetched from server
  let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(
    relyingPartyIdentifier:"example.com")
  let request = provider.createCredentialAssertionRequest(
    challenge: challenge)

  let credentialIDs: [Data] = … // Fetched from server for provided userName
  request.allowedCredentials = credentialIDs.map(
    ASAuthorizationPlatformPublicKeyCredentialDescriptor.init(credentialID:))

  let controller = ASAuthorizationController(authorizationRequests: [request])
  controller.delegate = self
  controller.presentationContextProvider = self

  // Start the request
  controller.performRequests()
}
```

- Apple platforms always require UV when biometrics are available, always use `userVerification: "preferred"`
- Using passkeys on the web
    - Make AutoFill requests early
    - Modal requests require a user gesture
    - Passkeys replace Safari’s legacy platform authenticator

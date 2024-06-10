# Introducing Car Keys

You can now store car keys on iPhone or Apple Watch. You no longer have to bring your key fob to unlock and start your car. And with digital keys, it's easy to share them with family or friends, and manage keys remotely.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10006", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



For specifications refer to the [Car Connectivity Consortium][CCC].

## What's Car Keys?

- With Car Keys you can lock, unlock and start supported cars via iPhone and Apple Watch.
- Securely stored on device and iCloud
- Shareable and manageable remotely
- Supports multiple radio technology: NFC (for now), Ultrawide-band soon
- Designed to work without a network connection
- Works with Power Reserve (aka even when your device is out of battery and switched off)

## Car compatibility

Three requirements:

- Owner pairing
- Transactions
- Server interfaces

### Owner pairing

Required to setup (additional) car keys on the iPhone.

Steps:

1. the user must prove ownership of the car (this step is up to the automaker)
2. initiate pairing (via app or email by automaker)
3. place iPhone nearby the car radio sensor
4. Car key appears in Wallet.app

### Transactions

- Used to lock. unlock, and start the car engine
- Cars are required to provide radio sensors in the door handle and dashboard
- Express mode
  - makes the transaction work without user identification first (face/touch id or password)
  - on by default, the user can turn it off

- completely offline
- No data is sent to Apple (a.k.a Apple doesn't know when the user uses the car)

### Service interfaces

- required for car key sharing and management
- Share keys via Messages.app
- The car can be offline when the sharing happens
- Fully encrypted: Apple has no info who the user shares the key with
- Automakers can provide different access levels to different car keys, for example to limit the car speed


## Key Management

- the owner can create, manage, revoke, and share keys on iPhone
- some functionality can be added in the dashboard
- car keys removed on iPhone stop working immediately, even if the device is offline
- iCloud lost mode temporarily suspends car keys on iPhone or Apple Watch (by locking the car key applet on the Secure Element)
- when purchasing a new phone, the owner needs to pair the new device to the car and all previous keys will be automatically transferred, shared keys will keep working seamlessly

## System Architecture

- All key management is integrated in iOS, doesn't relay on automakers
- keys are created and stored on Secure Element (NFC/ApplePay chip)
- use AES and Elliptic curve (EC) cryptography
- Offline design based on PKI (Public Key infrastructure)
- Automaker TSM (Trusted service manager) not required

### Key lifecycle

- Elliptic curve private/public key created in the device's Secure Element
- The private key never leaves the Secure Element
- The public key is exported in a X.509 certificate for authenticity certification
- A Secure Element Applet implements the car key:
  - stores key pair and binds it to the car
  - implements transactions and manages secure mailboxes which store key attestations and security tokens that prevent a revoked car key from being re-activated
  - Mailboxes also support immobilizer tokens for compatibility with existing car architectures
  - For memory efficiency all car keys are hosted in a single applet instance

[CCC]: http://carconnectivity.org
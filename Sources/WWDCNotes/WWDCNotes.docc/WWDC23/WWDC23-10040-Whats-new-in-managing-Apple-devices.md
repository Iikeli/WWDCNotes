# What’s new in managing Apple devices

Learn about the latest management capabilities for iOS, iPadOS, and macOS. Discover how you can streamline the setup experience with enhancements to automated device enrollment and a new return-to-service option for iOS and iPadOS devices. We’ll share how to use your identity provider in even more places on macOS and show you how Apple Configurator can help automate tasks. 


@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10040", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(MarcoEidinger)
   }
}



This sesson recommends in-deepth sessions about

- **Managed Apple IDs** received updates to Continuity, Apple Wallet, and iCloud Keychain. (WWDC23 "Do more with Managed Apple IDs")
- Businesses can now go passwordless with **passkeys**. (WWDC23 "Deploy passkeys at work" to learn more)
- **Declarative device management** has been significantly enhanced and supports new ways to deploy applications, certificates, and on macOS, even manage common service configuration files. (WWDC23 "Advances in declarative device management")
- **Apple Watch enrollment** is now possible, supporting even more institutional use-cases. (WWDC23 "Discover watchOS management")

This session focus is on additions for

- device management on **macOS**
- device management on **iOS and iPad**

# macOS

## Automated Device Enrollment

- Enforce FileVault
  - Enable during Setup Assistant
  - Show recovery key
  - Escrow to MDM
- Require a minimum OS Version
- Enforced Automated Device Enrollment

## Plattform SSO

- Platform SSO in System Settings

## Local account creation

- Create local user account just-in-time
- Enabled by the new `UseSharedDeviceKey`
- Requires the device to be
  - Online
  - At login window with FileVault unlocked
  - Managed by an MDM supporting Bootstrap Tokens
- Username and passwords or SmartCards 

## SmartCards
- authenticate at the login window and screen saver
  
## Group management and network authorization

- Standard, administrator, or group defined permissions
- Mapping of Identity Provider groups
- Non-local users at authorization prompts
- Exceptions:
  - Current user
  - SecureToken or ownership

## Password management

- Password policies with regular expressions
- Password compliance management

### Password compliance management
- Verify compliance after a password has been set
- Notification is shown during an active log-in session
- Password change notification prompted on next login

## System Settings Management

- Apple ID login and Internet Accounts
- Adding local-user accounts
- Device name
- Fingerprints for Touch ID
- Individual sharing services
- Siri
- Startup disk
- Time Machine

## Managed Device Attestation on macOS

- Devicelnformation attestation
ACME attestation
- Supports hardware-bound keys
  - Stored in data protection keychain
  - VPN, 802.1x, Kerberos, Exchange, MDM

## New attestation properties

- SIP status (Apple silicon Macs onlv)
- Secure Boot status (Apple silicon Macs only)
- Third-party kernel extensions allowed (Apple silicon Macs only)
- LLB version
- OS version
- Software Update Device ID
- Secure Enclave Enrollment ID

## Summary of macOS management updates

### Configuration profile updates

- Managed Device Attestation
- Hardware-bound private keys with ACME
- Platform SO supports local-user creation
- Support regex in passcode configuration
- Configure automatic login
- Configure built-in relay network extension
- Define order of transparent proxy extensions

### Automated Device Enrollment
- Enforce FileVault
- Require a minimum OS Version
- Enforced Automated Device Enrollment

### New MDM queries

Devicelnformation
(ModelNumber, Battery Level, EACSPreflight)

### New restrictions

- allowAccountModification 
- allowAssistant 
- allowCloudFreeform 
- allowDeviceNameModification 
- allowFingerprintModification 
- allowLocalUserCreation 
- allowRemoteAppleEventsModification 
- allowARDRemoteManagementModification
- allowStartupDiskModification 
- allowTimeMachineBackup
- allowBluetoothSharingModification
- allowFileSharingModification
- allowInternetSharingModification
- allowPrinterSharingModification

### Declarative device management

#### New configurations

- App management
- Certificate
- Config file
- Passkey
- Screen sharing
- Software update

#### New status items

- Background tasks
- Device model
- FileVault status
- Installed apps
- Installed certificates
- Software update

## Managed Mac apps
- Package can contain multiple applications
- Multiple applications are manageable
- MDM can remove individual applications
- Content outside /Applications is not managed

# iOS and iPadOS

## Return to Sender

- New `ReturntoService` dictionary in `EraseDevice` command
  - Provide Wi-Fi settings
  - Include an enrollment profile
- Previous language and region settings get applied

## Easy student sign-in
Requirements:
- Teacher and student in the same Apple School Manager location
- Local proximity of the devices
- Students have authorized the teacher on personal devices

## Enhancements for Shared iPad

- `AwaitUserConfiguration` allows you to fully configure a device after login
- `SkipLanguageAndLocaleSetupForNewUsers`
- Configure quota for temporary users on Shared iPad

## Private cellular networks

- iPhone and iPad support private LTE, standalone and non-standalone
5G networks
- Power efficient activation of private-network SIM based on geolocation
- Intelligent selection between private and public-network SIMs
- Option to prefer cellular over Wi-Fi

## 5G network slicing

- Configure managed apps to use specific 5G network slice
- Slice name for `CellularSliceUUID` to be defined by the carrier
- Mutually exclusive with VPN configurations

## Relay network extension

- Access enterprise resources without a VPN
- `com.apple.relay.managed` payload type and `NERelayManager` API
- Per-app, per-domain, or default route configurations
- Compatible with iCloud Private Relay

## Summary of iOS and iPadOS management updates

###  Configuration profile updates

- Configure private network geolocations
- Prefer cellular over Wi-Fi
- Assign managed apps to a 5G slice
- Configure built-in relay network extension
- Remove never from inactivity settings with User Enrollment
- 802.1X support for ethernet connections (incl. tvOS)
- Configure quota for temporary users on Shared iPad

### Automated Device Enrollment

### Declarative device management

- Require minimum OS version

#### New configurations

- App management
- Certificate
- Passkey
- Software update

#### New status items
- Device model
- Installed apps
- Installed certificates
- Software update

### New MDM commands and queries

- Return to service
- Show model number
- AwaitUserConfiguration on Shared iPad
- Skip language and locale selection for new users

### New app attributes

- TapToPayScreenLock

## iOS/iPadOS deprecations and supervision changes

### Deprecations
- APN payloads
- Top-level cellular keys in Devicelnformation
### Supervision
- allowAutoUnlock
- allowSharedStream
- allowInAppPurchases
- safariAllowJavaScript
- safariAllowPopups
- safariAcceptCookies
- allowFingerprintForUnlock
- allowSpotlightInternetResults
- allowGlobalBackgroundFetchWhenRoaming
- llowBookstoreErotica
- ratingApps
- ratingTVShows
- ratingMovies
- allowExplicitContent

### Supervision and personal Apple ID
- allowCloudPhotoLibrary
- allowCloudDocumentSync
- allowActivityContinuation
- allowCloudPrivateRelay

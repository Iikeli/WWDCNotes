# All About Notarization

Notarization is all about identifying and blocking malicious Mac software prior to distribution, without requiring App Review or the Mac App Store.  Introduced last year and already widely adopted by Mac app developers, this is your opportunity to take an in depth tour of Notarization workflows and find out what’s new with the Notarization service.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/703", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- **I highly recommend watching this session if you ever run into any problems regarding notarization** since this is a complex topic and this video is packed of useful examples
- **Notarization** identifies and blocks malicious software prior to distribution. It is **NOT** an App Review!
- comes on top of your Developer ID - no new registration
- **Notary Service** performs automated security checks
- **Process** Local Development > Distribution Signing > Notarization Attachment > Distribution via Website
- **On App Download** the Notarization attached to your app is checked by Notarization Service. Gatekeeper permits/denies installation.
- **Benefits**
  - prevents the developer from shipping a malicious dependency
  - apps with the hardened runtime are more secure by default
  - users are ore likely to try and download new software
  - audit trail of software notarized by your Developer ID account 

- Software **signed on or after June 1, 2019** must adopt
  - complete and correct signing
  - the hardened runtime

- **Complete and Correct Signing** involves
  - signing **everything** (`Bundles`, `Mach-Os`, `Installer packages`) with your Developer ID Application Certificate and include a secure timestamp
  - Executables must opt-in  the hardened runtime
  - Sign `Installer Packages` with Your Developer ID Installer Certificate
  - Sign `Disk images` with Application Certificate and include secure timestamp
  - Enable `Xcode Automatic Codesigning` - it does it for you

- **Hardened Runtime** extends macOS system integrity protection features to your apps
  - **Runtime code signing enforcement**
    - configurable via entitlements
    - Adopt via `codesign --sign "Developer ID" --timestamp --options runtime My.app`
    - Verify via `codesign --display --verbose=2 My.app` and make sure `runtime` is printed next to `flags`
    - Look into [12:04](https://developer.apple.com/wwdc19/703/?time=725) for detailed description
    - Look into [12:22](https://developer.apple.com/wwdc19/703/?time=742) if your app crashes because you use JIT
    - Look into [13:54](https://developer.apple.com/wwdc19/703/?time=831) if  your app crashes because you patch system frameworks - **don't do this**
    - If your app crashes on auto-update: create a new file when you update a signed file

- 
  - **Library Validation** 
    - protects your app from code injection and dylibs hijacking
    - prevents loading unsigned or adhoc-signed code
    - Detailed solutions for common issues can be fount at [16:00](https://developer.apple.com/wwdc19/703/?time=957), e.g. `App loads plugins from other devs in-process`, 

- 
  - **DYLD Variable Environment Protection**
    - can inject libs and modify framework and lib search path - useful for testing
    - Blocks `DYLD_LIBRARY_PATH`, `DYLD_INSERT_LIBRARIES`, `DYLD_FRAMEWORK_PATH` by default
    - Don't use DYLD environment variables when shipping to customers
    - You can use `com.apple.security.get-task-allow` entitlement during debug build

- 
  - **Debugging Protection**
    - disables debugging hardened processes by default
    - You can use `com.apple.security.get-task-allow` entitlement during debug build to get around this - Xcode does it automatically

- 
  - **Protected Resource Access**
    - App needs to declare its intent to access protected resources, e.g. location, photos, contacts, ...
    - settable via entitlements - see [20:46](https://developer.apple.com/wwdc19/703/?time=1225)
  - Use only entitlements really needed
  - Set those entitlements only for processes that need them
  - Set resource-access entitlements only on main bundle; get inherited by other bundles

- Notarization can be done easily via the Archive menu from within Xcode
- `xcrun altool --notarize-app ...` to submit an app via command line and check via `xcrun altool --notarization-info <request_id_from_submission> …`  for the current status
- Use `xcrun altool --notarization-history …` to get on overview of all the software submitted on your account

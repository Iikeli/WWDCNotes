# Advances in macOS Security

We are on a journey to continuously improve macOS security, with a particular focus on preventing malware and protecting user data. Join us on the next step and learn more about what’s new in Gatekeeper—for keeping malware out of macOS—as well as new protections that help keep users’ data and activity under their control.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/701", purpose: link, label: "Watch Video (40 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



Defense in depth: there isn’t any single layer that can always perfectly protect you, so there are multiple layers of security, so if any single layer fails, that doesn’t defeat the whole security of the system. Layers can delay the advance of the attacker, reduce the attack surface, create “choke points” that are easier to defend.

Gatekeeper: designed to protect users from running malicious software, while allowing them to use the software they choose.

What does Gatekeeper check:

- does the app contain any known malicious content?
- has the software been tampered with since it was signed?
- does it meet the security policy configured on the computer?
- first launch prompt ⭢ does the user actually want to run this?

On Mojave, Gatekeeper runs the check on the 1st launch of quarantined software launched via Launch Services.

Quarantine - a technology on macOS for marking files that arrived from some external source (website, airdrop, iMessage, email)

- includes metadata about where the file came from
- opt-in - the app has to opt-in to this, so e.g. when apps download their own updates they are usually not quarantined, except for sandboxed apps

Launch Services - a framework for finding and launching apps on macOS, used when launching apps from Finder, [`NSWorkspace`](https://developer.apple.com/documentation/appkit/nsworkspace), document handlers etc.

What does _not_ use Launch Services: [`NSTask`](https://developer.apple.com/documentation/foundation/nstask), [`NSBundle`](https://developer.apple.com/documentation/foundation/nsbundle)/`dlopen`, `exec`/`posix_spawn`.

In macOS Catalina:

- all new software must be notarized to pass Gatekeeper
- all software is checked when first launched, even when launching through those non-LaunchServices methods
- all software (even not quarantined) is checked for malicious content on every launch

> *"_You can always choose to run any software on your system_"* - there will always be a way to run a specific piece of software that you want to run
>
> *“We want to make macOS just as secure as iOS, while still maintaining the flexibility that you’ve come to expect from your Mac”*

Platform security is increasingly reliant on validity of code signatures; that means if code has no signature, it’s impossible to detect tampering.

In a future version of macOS, unsigned code will not load by default, so:

- sign and notarize all software
- don’t modify signed applications and bundles
- handle failures when loading libraries


## Privacy changes:

Requires user confirmation for:

- screen recording
- keyboard input monitoring

Requires confirmation for access to:

- Desktop, Documents, Downloads
- iCloud Drive and third-party cloud storage
- Removable and network volumes

But:

- \*not\* required for creating new files, only for reading existing files
- tries to understand intent, e.g. doesn’t ask if user double-clicked a file in Finder, or drag&dropped it, or used an open/save panel
- declare handled [`CFBundleDocumentTypes`](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/20001431-101685) with `NSIsRelatedItemType` to e.g. automatically have access to a subtitles file when opening a movie file

Purpose strings are accepted, but not required (`NSDesktopFolderUsageDescription` etc.).

Open and save panels always run out of process. Be careful with:

- [`panel(_:userEnteredFilename:confirmed:)`](https://developer.apple.com/documentation/appkit/nsopensavepaneldelegate/1524630-panel)
- [`panel(_:validate:)`](https://developer.apple.com/documentation/appkit/nsopensavepaneldelegate/1535141-panel)
- [`panel(_:didChangeToDirectoryURL:)`](https://developer.apple.com/documentation/appkit/nsopensavepaneldelegate/1527117-panel)

Checking for readability without triggering a consent dialog: [`isReadableFile`](https://developer.apple.com/documentation/foundation/filemanager/1418292-isreadablefile), [`isWritableFile`](https://developer.apple.com/documentation/foundation/filemanager/1416680-iswritablefile), [`access()`](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man2/access.2.html).

Apps and other binaries that have previously been denied access to some kind of directory now appear automatically in the "Security & Privacy" access list, unchecked.

Full disk access now required for access to Trash (except files that your app has moved there).

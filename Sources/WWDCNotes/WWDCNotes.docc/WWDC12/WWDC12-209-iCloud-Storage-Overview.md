# iCloud Storage Overview

iCloud Storage enables apps to store documents and settings across iOS and OS X. Discover how iCloud Storage works, learn about the latest advancements in development and debugging for iCloud and the Key-Value Store, and learn how your app can use iCloud to store documents and settings today.

@Metadata {
   @TitleHeading("WWDC12")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc12/209", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



iCloud Storage APIs allow you to store your app’s data in iCloud. System services sync your data automatically even when your app isn’t running.

3 different types of storage:

- key-value storage - simple storage for things like preferences, game state (“*it’s so simple, we actually don’t have another session talking about it*")
- document storage - a filesystem in the cloud scoped for your application, where you can store any kinds of files and folders, synced between devices; ideal for productivity apps like iWork
- Core Data storage - an extension for Core Data that lets you store Core Data databases in the cloud [note: deprecated in 2016]

What iCloud handles for you:

- account setup - users don’t need to create a new account for your service, they already have an iCloud account
- APIs for your apps integrated into the OS X/iOS SDKs
- server code you don’t have to write, for things like load balancing, replication, backup and recovery
- the personnel handling the servers, support etc.

> “If you happen to know some friends who like writing server code that scales to hundreds of millions of users, send them our way - we’re hiring” ;)


## How it works

### Key-value storage:

- Accessed through [`NSUbiquitousKeyValueStore`](https://developer.apple.com/documentation/foundation/nsubiquitouskeyvaluestore)
- Lets you put simple plist-type values into the cloud
- It talks to a key-value service running on the device which talks to iCloud on your behalf

### Document storage:

- Use [`UIDocument`](https://developer.apple.com/documentation/uikit/uidocument) (iOS) / [`NSDocument`](https://developer.apple.com/documentation/appkit/nsdocument) (OS X) / [`UIManagedDocument`](https://developer.apple.com/documentation/uikit/uimanageddocument) (Core Data)
- You can also use lower level file storage APIs like NSFileCoordination, [`NSFilePresenter`](https://developer.apple.com/documentation/foundation/nsfilepresenter), [`NSMetadataQuery`](https://developer.apple.com/documentation/foundation/nsmetadataquery)

All these APIs (also the Core Data iCloud API) talk to OS’s document service which talks to the iCloud for you. You never actually interact with the iCloud servers yourself, you just use these APIs in the SDK and system services.


## How to enable iCloud in your project

- your app needs to be distributed through the App Store [note: no longer true for Mac apps]
- enable the relevant entitlement: `com.apple.developer.ubiquity-kvstore-identifier` and/or `com.apple.developer.ubiquity-container-identifiers`


## Working with Key Value Storage

[`NSUbiquitousKeyValueStore`](https://developer.apple.com/documentation/foundation/nsubiquitouskeyvaluestore):

- lets you store simple plist values - strings, numbers, booleans, dictionaries and arrays of those
- similar API to [`UserDefaults`](https://developer.apple.com/documentation/foundation/userdefaults)
- simple conflict resolution: if two devices independently change or add a value for a given key, the latest change wins
- works even if iCloud isn’t configured - in this case it just acts as local user defaults that don’t sync

Improvements from last year’s initial release:

- increased capacity - now up to 1024 keys, up to 1 MB per application (not part of the total user quota); it’s fine to have just one key of 1 MB
- improved responsiveness (allows around 15 requests every 90 seconds)

How to set up:

```objc
// get a reference to the store
NSUbiquitousKeyValueStore *store = [NSUbiquitousKeyValueStore defaultStore];

// observe changes
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
[center addObserver:self
           selector:@selector(kvStoreDidChange:)
               name:NSUbiquitousKeyValueStoreDidChangeExternallyNotification
             object:nil];

// ask for any changes since the last launch
[store synchronize];
```

Making changes:

```objc
[store setObject:someObject forKey:@“someKey”];
[store setBool:YES forKey:@“someOtherKey”];

[store synchronize];
```

It’s recommended to also store another copy of the same data locally, so that you can do conflict resolution manually in case if just keeping the latest change isn’t always the right strategy for your app.

Handling notifications about a change:

```objc
- (void)kvStoreDidChange:(NSNotification *)notification {
    NSDictionary *userInfo = [notification userInfo];

    // get change reason
    int reason = [userInfo objectForKey:NSUbiquitousKeyValueStoreChangeReasonKey];
    NSArray *changedKeys = [userInfo objectForKey:NSUbiquitousKeyValueStoreChangedKeysKey];

    // … store values locally, do conflict resolution etc.
}
```


## Working with document storage

Apart from your app’s container, each app has a separate “ubiquity container” (or iCloud container). The app can put any files inside that container, and whatever is put there is synced with other devices via iCloud, kind of like Dropbox.

When you put a file in the ubiquity container, the file is broken into chunks and the chunks that are new or modified are uploaded to iCloud - so if you only change a few bytes of the file, most of it doesn’t need to be uploaded again.

Metadata about the file is uploaded first - info about the file name, type, size etc. So every device knows about each file that’s uploaded, but it doesn’t necessarily have to download each file - the decision is made independently on the device depending on the platform and settings: OS X usually downloads all files if it has space, iOS only downloads files on demand. Once a file is downloaded to the device, all later changes are also automatically synced.

Local peer to peer communication is used if possible - e.g. if you have a file on the Mac, your iPhone will copy it from the Mac instead of the iCloud network.

Automatic conflict resolution - if a file is edited on two devices in parallel, the system picks a winner automatically, but your application gets access to both versions and can override it if needed.

URL publishing: you can make the current version of the document public and available through a URL which you can share with others (if it’s changed later in the iCloud, the URL still downloads that previous version). URLs are not permanent, they expire after some time.


### Detecting an iCloud account:

Unlike key-value storage, using this API requires the user to have an iCloud account. You can check for an “Ubiquity Identity Token” to see if they have an account configured.

The token is anonymous so it doesn’t tell you anything about the user, but it will change if the user switches to another account (you also get a notification then). The token is also unique to your app and to this specific device, so the same app will get a different token from that user on another device.

```objc
id token = [[NSFileManager defaultManager] ubiquityIdentityToken];
if (token) {
    // cache the token
    // the next time the app launches, check if it has changed
}

// register for the identity changed notification
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
[center addObserver:self
           selector:@selector(handleUbiquityIdentityChanged:)
               name:NSUbiquityIdentityDidChangeNotification
             object:nil];
```

When the account changes, clear any local caches specific to this account and refresh the UI.

To access the ubiquity container, you ask for the container URL - note that the container will be created on demand the first time you ask for it; this should ideally not be called on the main thread.

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    containerURL = [fileManager URLForUbiquityContainerIdentifier:nil];
});
```

Types of documents you can store in the document storage:

- normal files, also symlinks
- directories of files
- packages - bundles of files that act as a single document
- Core Data stores

File extended attributes are also synced.

Watch out for filesystem case sensitivity issues - users running on a Mac might have a case-sensitive filesystem.

For packages, iCloud updates only the files from a package that have been changed, but it handles updates to the whole package atomically, so you will not get a package in an inconsistent state.


## Core Data [deprecated]

Useful for so-called “shoebox style applications”, like iPhoto or iTunes, which work with a single database in app’s own format.

The Core Data store remains local and only change logs are uploaded to iCloud. It's not recommended to use binary and XML stores, because in those cases every change modifies the whole file (only use those for small data sets that don’t change often) - use SQLite stores for iCloud sync instead.

[`UIManagedDocument`](https://developer.apple.com/documentation/uikit/uimanageddocument) - a subclass of [`UIDocument`](https://developer.apple.com/documentation/uikit/uidocument) for managing Core Data stores that supports syncing them with iCloud. Note: [`NSPersistentDocument`](https://developer.apple.com/documentation/appkit/nspersistentdocument), the AppKit equivalent, does not support iCloud.


## Designing your document format for iCloud

- Design with network efficiency in mind - don’t write a lot of changes very often
- Keep in mind any possible differences between platforms
- Plan for future app upgrades - include version number in the format and keep compatibility if possible
- Beware of sync loops - when one instance of the app receives a change, merges it and writes back the result, and the other side does the same and triggers a change in the first copy again (“And you have two versions of the app playing ping-pong with the user’s iCloud account”)
- Avoid making rapid changes to the file, e.g. updating some position tag while the user is scrolling the document
- Don’t put the last open date into the document itself, so that opening it doesn’t count as making a change
- Use iCloud for user data only: don’t put any caches, temporary files or auto-generated content there
- Think about privacy when allowing the user to publish a document: don’t include any sensitive info or things they might not be aware they’re publishing (e.g. undo history) in the publicly accessible view


## APIs

- [`NSFileManager`](https://developer.apple.com/documentation/foundation/filemanager)
- [`NSFileCoordinator`](https://developer.apple.com/documentation/foundation/nsfilecoordinator) & [`NSFilePresenter`](https://developer.apple.com/documentation/foundation/nsfilepresenter)
- [`NSMetadataQuery`](https://developer.apple.com/documentation/foundation/nsmetadataquery) - use a live metadata query to be notified of new files and changes before the file contents are downloaded
- [`NSDocument`](https://developer.apple.com/documentation/appkit/nsdocument) & [`UIDocument`](https://developer.apple.com/documentation/uikit/uidocument)
- [`UIManagedDocument`](https://developer.apple.com/documentation/uikit/uimanageddocument)

[`NSDocument`](https://developer.apple.com/documentation/appkit/nsdocument)/[`UIDocument`](https://developer.apple.com/documentation/uikit/uidocument) handle most of the integration with iCloud for you:

- using the ubiquity container
- coordinating with the OS
- tracking files and their versions
- resolving conflicts

### Tips:

- subclass native document types
- use the default auto-save behavior
- if the app provides a prepopulated Core Data store on first launch, use a migration instead of copying a packaged store file
- track documents using [`NSMetadataQuery`](https://developer.apple.com/documentation/foundation/nsmetadataquery)
- if it makes sense for your app, manually control conflict resolution ([`UIDocumentStateInConflict`](https://developer.apple.com/documentation/uikit/uidocument/state/1619972-inconflict)); avoid user involvement if possible

### Tips for debugging:

- test with multiple devices
- monitor network traffic
- use Airplane Mode to create conflicts artificially
- there will be a configuration profile available that makes the document service log additional messages
- [developer.icloud.com](https://developer.icloud.com) - a web tool that shows you all iCloud storage on your account from various apps [note: not available anymore, replaced with CloudKit dashboard, but it only shows CloudKit data]

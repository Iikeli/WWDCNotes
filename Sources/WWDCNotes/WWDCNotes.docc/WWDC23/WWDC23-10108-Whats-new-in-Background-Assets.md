# What’s new in Background Assets

Waiting is no fun! Discover how Background Assets can help your app download content before it even launches. We’ll show you how to integrate Background Assets into an existing app, explore when to use essential or non-essential assets, and learn how to make debugging your extension a breeze.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10108", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(srujanc)
   }
}



# Background Assets

With the [Background Assets framework](https://developer.apple.com/documentation/backgroundassets?language=objc), you can improve the experience of your app or game by reducing or eliminating the time people have to wait while your app downloads any required assets at first launch

> Add a Background Assets extension to your app’s target, and let the system notify that extension about an app installation or subsequent update. Then use the download manager to schedule background downloads of required content from your servers or content delivery network (CDN), and have those downloads finish even when the app isn’t running Check for updated content when the system periodically launches the extension (dependent on app usage) and, when content is available, schedule it for immediate download. 

[!Important] Use the framework only to download additional assets for your app; don’t use it for any other purposes. For example, don’t collect or transmit data to identify a user or device or to perform advertising or advertising measurement.

The framework leverages [ExtensionKit](https://developer.apple.com/documentation/extensionkit) and common types like URLRequest.


# What’s new in Background Assets

In this New Background Assets approach, Users can be able to download assets while installing the app or updating unlike old version while launching the app. This means that downloads are completely integrated into the iOS Home Screen, macOS Launchpad, and the App Store. To the end user, the download of assets appear as if the app is currently still being downloaded from the App Store. This also means that while essential downloads are in-flight, app cannot be launched by the user. All the user can do is cancel or pause installation.

### Changes required

Below are the essential changes that required compared to background assets that were introduced back year.

``` swift

/// Old Changes in info.plist

|Key|Type|Description|
|-|-|-|
|BAInitialDownloadRestrictions|Dictionary|The restrictions that apply to the set of assets that download prior to first app launch.
|BADownloadAllowance|Number|The combined size of the initial set of non-Essential asset downloads. Stored inside the BAInitialDownloadRestrictions dictionary.
|BADownloadDomainAllowList|Array|Array of domains that can assets can be downloaded from prior to first app launch. Stored inside the BAInitialDownloadRestrictions dictionary.
|BAMaxInstallSize|Number|The combined size (in bytes) on disk of the Non-Essential assets that download immediately after app installation.
|BAManifestURL|String|URL of the application's manifest.


/// New Change in info.plist

|Key|Type|Description|
|-|-|-|
|BAInitialDownloadRestrictions|Dictionary|The restrictions that apply to the set of assets that download prior to first app launch.
|BADownloadAllowance|Number|The combined size of the initial set of non-Essential asset downloads. Stored inside the BAInitialDownloadRestrictions dictionary.
|**BAEssentialDownloadAllowance**|Number|The combined size (in bytes) of the initial set of Essential asset downloads, including your manifest. Stored inside the BAInitialDownloadRestrictions dictionary.
|BADownloadDomainAllowList|Array|Array of domains that can assets can be downloaded from prior to first app launch. Stored inside the BAInitialDownloadRestrictions dictionary.
|BAMaxInstallSize|Number|The combined size (in bytes) on disk of the Non-Essential assets that download immediately after app installation.
|**BAEssentialMaxInstallSize**|Number|The combined size (in bytes) on disk of the Essential downloads that occur during app installation.
|BAManifestURL|String|URL of the application's manifest.

```

[!Important] The above keys are not only essential but are also necessary in order to submit the app to AppStore.

There are two new keys that are required to support Assential Assets BAEssentialDownloadAllowance and BAEssentialMaxInstallSize. The essential download allowance is represented in bytes and defines an upper bound on how large the sum of all of your essential assets will take to download. It's important to try to get this number as close as possible to the size of the essential assets you enqueue so that download progress is smooth for the user when they install your app. The other new key, BAEssentialMaxInstallSize, represents the maximum size of those assets extracted onto the user's device.

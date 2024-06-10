# Get started with privacy manifests

Meet privacy manifests: a new tool that helps you accurately identify the privacy practices of your app’s dependencies. Find out how third-party SDK developers can use these manifests to share privacy practices for their frameworks. We’ll also share how Xcode can produce a full privacy report to help you more easily represent the privacy practices of all the code in your app.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10060", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(MarcoEidinger)
   }
}



# Privacy manifests

Privacy manifests are a new way for third-party SDK developers to provide information about their privacy practices. This information helps app developers accurately represent privacy in their app.

Third-party SDK developers can create a new privacy manifest right from the Xcode navigator, by creating a file named "PrivacyInfo.xcprivacy".

![Privacy Manifests][privacymanifest]

[privacymanifest]: WWDC23-10060-privacymanifest

This file is a property list that declares what data types the SDK collects, how each data type is used, whether they are linked to the user, and whether they're used for tracking as defined by the App Tracking Transparency policy.

![Privacy Manifests][examplemanifest]

[examplemanifest]: WWDC23-10060-examplemanifest

Taking a closer look at this manifest, the Example Sharing SDK is listed as not tracking. It collects name and user ID, linked to the user, for both app functionality and product personalization. It also collects photos or videos, linked to the user, for just app functionality.

[Apple Documentation: Describing data use in privacy manifests](https://developer.apple.com/documentation/bundleresources/describing_data_use_in_privacy_manifests/)

# Privacy report

When you're building your app to submit to the App Store, Xcode 15 can aggregate all the privacy manifests in your app's project, and produce a privacy report that summarizes the declared data uses. To view this, go to Xcode Organizer, show the context menu for an archive, and select "Generate Privacy Report."

![Generate Privacy Report][generateprivacyreport]

[generateprivacyreport]: WWDC23-10060-generateprivacyreport

The privacy report is a PDF and easy to use. It is organized in a similar way to Privacy Nutrition Labels. So you can easily reference this report when you provide your app's privacy details in App Store Connect.

![Privacy Report][privacyreport]

[privacyreport]: WWDC23-10060-privacyreport

This helps you review, understand, and describe the privacy practices of your app and its dependencies.

# Tracking domains

Privacy manifests, that declare tracking, may include tracking domains. This helps app developers and third-party SDK developers to avoid tracking people without their permission.

![Specify tracking domains in your privacy manifest][trackingdomain]

[trackingdomain]: WWDC23-10060-trackingdomain

In cases when a user has not provided tracking permission, iOS 17 automatically blocks connections to tracking domains that have been specified in any privacy manifest included in your app.

# Required reason APIs

Fingerprinting, using signals from the device to try to identify the device or user, is never allowed. There are existing APIs in our platforms that have the potential of being mis-used for fingerprinting. However, these APIs also provide powerful user experiences when accessed appropriately. To support important use cases that benefit the user while avoiding fingerprinting, there is a new category of APIs called Required reason APIs. Apple begun grouping these APIs into categories. For each category, there is a list of approved reasons to access these APIs, based on their use cases. For example, one Required reason API is `NSFileSystemFreeSize,` which indicates the amount of free space on the file system. One of its approved reasons supports using this API to check whether there is sufficient disk space before writing files to disk.

The list of Required reason APIs and approved reasons, including any future updates, will be published in the Apple developer documentation during the course of 2023. The total number of Required reason APIs will be small, but it is likely that developers use one or more of them. If you have a use case for an API category that is not already covered by an approved reason, and the use case directly benefits the user, the documentation will link to a feedback form where you can let us know.

To protect users from possible fingerprinting, apps and SDKs are allowed to access the Required reason APIs only for the approved reasons.

Apple will publish additional information later this year, including:

- A list of “required reason” APIs for which an allowed reason must be declared
- A developer feedback form to suggest new reasons for calling covered APIs

# Next Steps

App & third party SDK developers are highly encouraged to adopt package manifests. Adoption is optional unless your
- app is using a privacy-impacting SDK or 
- SDK is classified as a privacy-impacting SDK

![Privacy-impacting SDKs][privacyimpactingsdks]

[privacyimpactingsdks]: WWDC23-10060-privacyimpactingsdks

Apple will a list of privacy-impacting SDKs later this year.

Starting in Fall 2023, App Store will check if new and updated apps include a library from a privacy-impacting SDK. If the privacy-impacting SDK does not include a signature and privacy manifest, Apple will send an informational email to the app developer. 

Apple will also send informational emails for apps that access Required Reason APIs without declaring approved reasons.

Starting in Spring 2024, these will be expected and become part of App Review. You'll need to address any issues before you can submit new and updated apps to the App Store.
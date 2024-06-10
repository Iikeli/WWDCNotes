# What's new in App Clips

Explore the latest updates to App Clips! Discover how we’ve made your App Clip even easier to build with improvements to the size limit as well as CloudKit and keychain usage. We’ll also show you how to use our validation tool to verify your App Clip and automate workflows for your advanced App Clip experiences using App Store Connect.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10097", purpose: link, label: "Watch Video (9 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- New size limit: Up to 15MB, uncompressed, thinned App Clip bundle (only when iOS 16 is minimum OS)
- New <kbd>Diagnostics</kbd> tool: 
  - go to <kbd>Settings.app > App Clips Testing > Diagnostics</kbd>
  - insert your app clips URL
  - the tool will check your URL's configuration and give you feedback if necessary

- CloudKit
  - New in iOS 16, App Clips can read data and resources stored in a CloudKit public database
  - To enable CloudKit, go to <kbd>Signing & Capabilities</kbd> of your App Clip target, and select the CloudKit container you want your App Clip to use
  - the public container can be a new one or one shared with the app
  - Still cannot write data to CloudKit
  - No private or shared database access
  - no Cloud documents or key-value store

- Keychain migration
  - Now your App Clip can simply store secure items in the keychain and they will be moved to your app when it's installed.
  - There's no longer need to migrate this data via a shared app group container
# Discover AppleSeed for IT and Managed Software Updates

With AppleSeed for IT, you can help your school or business test pre-release versions of Apple software and provide valuable feedback directly to Apple. We'll guide you through getting started in AppleSeed for IT and provide insight on how to file great feedback collaboratively within your organization using the new Teams feature in Feedback Assistant. You'll also learn more about Managed Software Updates in macOS Big Sur, including how to delay major updates or security and system files for employees’ machines while you certify the release on their systems.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10138", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Testing pre-release software

- All programs support iOS, iPadOS, macOS, tvOS, watchOS

- The Public Beta is available to all users who sign up at beta.apple.com
  - geared towards reporting livability or general use type issues
  - Updates in this program can be less frequent
  - Support for iOS, iPadOS, macOS, tvOS, watchOS

- The Developer Program is for app developers
  - geared towards testing new SDKs and make sure developers apps run on the new OS when it releases
  - More frequent updates
  

- AppleSeed for IT.

## The AppleSeed for IT program

- AppleSeed for IT is a seeding program focused towards IT professionals in enterprise and education
- Apple deploys a variety of software across all platforms for you to test
- Apple provides in-depth beta documentation and release notes that are focused on IT admin workflows

- Test plans and survey are provided to ensure the newest features work in your existing environment
- The surveys are so you can help Apple better understand that environment
- this is for you to provide your unique feedback on mission-critical apps and services that are important to your employees

- Apple is most interested in:
  - feedback about deployment blockers
  - issues that would prevent you from deploying the latest OS when we release it
  - regressions

### Get started

- create a Managed Apple ID
  - If you don't have one, talk to your Apple Business Manager or Apple School Manager administrator about generating one

- log in to [appleseed.apple.com](https://appleseed.apple.com)
- configure your test devices
  - you can push the configuration profile or the macOS Customer Beta Access Utility to any device you want to use for testing

- Within Apple Business Manager or Apple School Manager, ensure that roles you want to participate have `Participate in AppleSeed for IT` privilege enabled (By default, this privilege is enabled on all roles in Apple Business Manager)
- There must always be an AppleSeed for IT administrator in your organization
- The `Administer AppleSeed for IT` privilege is required for administrator roles and optional for people manager roles

## Filing feedback for your organization

- file immediately after the issue occurs with the device it occurs on as this will ensure the necessary logs are gathered
- include thorough steps to reproduce
- if relevant, attach a screen recording or a screenshot of the issue

### Feedback Assistant

- available on iOS, iPadOS, macOS and the [web](https://feedbackassistant.apple.com)
- designed to provide concise forms to capture the problem, to collect system logs, and diagnostics
- place for you to keep track of your feedback

- new this year: Teams
  - Teams allows members within an organization to work together on feedback with Apple
  - see feedback submitted by your peers
  - contribute to discussions with Apple
  - reassign feedback to team members
  - configured by Apple Business Manager or Apple School Manager for AppleSeed for IT, and in App Store Connect for the Developer Program
  - when filing a feedback, team members will be able to choose to file it as a team or as a personal feedback
  - personal feedback can move into the team via `more > Move to Team`
  - team administrator can reassign feedback to different team members, or close feedback assigned to others

- new this year: Multi-device diagnostic
  - initiate feedback from an iPhone or iPad
  - collect logs from multiple devices
  - must be signed into iCloud
  - when writing the feedback, tap `add attachment` and choose the other device whose diagnostic you'd like to submit

# Designing Notifications

Thoughtfully designed notifications are a powerful way to communicate timely information to people that they will find valuable and useful. Learn how you can design notifications people want to receive by making them beautiful, helpful, actionable, and respectful of their valuable time and attention.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/806", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Common Sense

The first part of the session is a recap of the other [notification][wwdc18710] [sessions][wwdc18711] with some extra common sense such as:

- when it is ok for an app to deliver quiet notifications, therefore without prompting the user for the notification authorization and the likes.
- Notifications should provide content and value to the user
- Notifications are not reasons to open your app (stop.spamming.)

## Rich notifications and Apple Watch

- Some features of our notifications are automatically translated to an Apple Watch notification for free, for example:
  - Image attachment
  - Notification title
  - Notification body
  - And quick actions
  - Ways we can customize the notification:
    - default:
      ![][1Image]
    - with customization:
      ![][2Image]

- 
  - Sash color (the light blue strip on the app name)
  - Background color (the background blur has customizable tint) 
  - Add in-line images/icons/videos

- (New in watchOS 5) Interactive notifications (like iOS 12), available if you have a watch app

[wwdc18710]: ../710/
[wwdc18711]: ../711/

[1Image]: WWDC18-806-1
[2Image]: WWDC18-806-2
# Meet Push Notifications Console

The Push Notifications Console is the best way to quickly test user notifications in your app. Learn how you can iterate on new ideas quickly by sending notifications directly from the console and analyze delivery logs to learn more about your pushes. We’ll also show you how to generate and validate tokens to successfully authenticate with Apple Push Notification service (APNs).

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10025", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(MarcoEidinger)
   }
}




The Push Notifications Console, a web-based tool with a variety of instruments for interacting with Apple Push Notification service (APNs), allows you to

- **send notifications** from the console to your device through APNS.
- **inspect delivery logs** to analyze why a message was not received.
- **validate and/or generate JWT** when using token-based authentication
- **validate device tokens**

Access to the tool: https://icloud.developer.apple.com/dashboard/notifications

## Send Notifications

The Console allows you to test many types of notifications and different attributes. You can

- specify the environment
- try different push types
- set the exact expiration
- try different priorities
- and send **any type of payload**.

![Send Notification][consolesendnotification]

[consolesendnotification]: consolesendnotification.png

## Delivery Log

Analyze why a notification was not received

As a notification travels through the APNs stack the events that reflect its delivery process are recorded. And now you can retrieve that information, using the header `apns-unique-id` that APNs returns when the notification is sent.

![apns-unique-id][uniqueid]

[uniqueid]: uniqueid.png

For example, you can identify a push notification was not yet delivered because the device has enabled Low Power mode.

Or notification can go to APNs storage if the device is offline or can be discarded if the app was removed from the device.

![Delivery Log][deliverylog]

[deliverylog]: deliverylog.png

## Generate JWT

Token-based authentication uses JSON Web Tokens for secure and efficient authentication between your provider server and APNs. It requires generating a token signed with a private key associated with your Apple Developer account. Private keys don't expire like certificates. As part of Push Notifications Console, there's now a tool that can generate an authentication token for you. 

![Generate JWT][jwtgenerator]

[jwtgenerator]: jwtgenerator.png

You can then use it to authenticate your requests against APNs. Keep in mind that the validity period of these tokens can not exceed one hour, so they need to be rotated periodically.

## Validate JWT

If you already have a token, but think it might not be working, you can validate the token.

![Validate JWT][jwtvalidation]

[jwtvalidation]: jwtvalidation.png

In the example above the validation result is telling me that the “issued at” claim is too old, which effectively means that the token has expired.

## Validate Device Token

Device tokens are used to specify the recipient when you send a notification. They are tied to a concrete environment and push type.

When you enter a token, you will get a response that will tell you which environment and push type the token is valid for, if any.

![Validate Device Token][devicetokenvalidation]

[devicetokenvalidation]: devicetokenvalidation.png
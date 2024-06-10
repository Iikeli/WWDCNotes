# Update Live Activities with push notifications

Discover how you can remotely update Live Activities in your app when you push content through Apple Push Notification service (APNs). We’ll show you how to configure your first Live Activity push locally so you can quickly iterate on your implementation. Learn best practices for determining your push priority and configuring alerting updates, and explore how to further improve your Live Activities with relevance score and stale date.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10185", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(srujanc)
   }
}



# Update Live Activities with push notifications

Update Live Activities remotely in an app when you push content through Apple Push Notification service (APNs). Here will explain how to configure Live Activity and push locally. you should be familiar with [ActivityKit](https://developer.apple.com/documentation/ActivityKit) and Live Activities

> Assuming that you've already know about how an App and server interacts with Apple Push Notifications Services(APNS). When a new Live Activity is started, ActivityKit will obtain a push token from Apple Push Notification service, or APNs for short. This push token is unique for each Live Activity you request. That's why your app needs to send it to your server before it can start sending push updates. Then, whenever you need to update the Live Activity, your server sends the push request using the token to APNs. Finally, APNs will send the payload to the device, and it will wake your widget extension to render the UI.

## Acitivity Framework

> With the ActivityKit framework, you can [Display a Live Activity to share live updates](https://developer.apple.com/documentation/activitykit/displaying-live-data-with-live-activities) from an app Lock Screen. Especially for apps that push the limit of notifications to provide updated information, a Live Activities can offer a richer, interactive and highly glanceable way for people to keep track of an event or activity over a couple of hours.

## Live Activity Type 

Live Activities display your app’s most current data on the iPhone or iPad Lock Screen and in the Dynamic Island, allowing people to see live information at a glance and perform quick actions that are related to the displayed information.


To support Live Activities:

1. [Create a widget extension](https://developer.apple.com/documentation/WidgetKit/Creating-a-Widget-Extension) if you haven’t added and make sure to select “Include Live Activity” when you add a widget extension target to your Xcode project. 

2. Add the Supports Live Activities entry in info.plist, and set its Boolean value to YES. Alternatively, open the Info.plist file as source code, add the NSSupportsLiveActivities key, then set the type to Boolean and its value to YES. If your project doesn’t have an Info.plist file, add the Supports Live Activities entry to the list of custom iOS target properties for your iOS app target and set its value to YES.

3. Add code that defines an ActivityAttributes structure to describe the static and dynamic data of your Live Activity.

4. Use the ActivityAttributes defined to create the ActivityConfiguration.

5. Add code to configure, start, update, and end your Live Activities.

6. Make Live Activity interactive with Button or Toggle as described in Adding interactivity to widgets and Live Activities.

7. Add animations to bring attention to content updates as described in Animating data updates in widgets and Live Activities.


## Construct the ActivityKit push notification payload

> To successfully update or end a Live Activity with an ActivityKit push notification, send an HTTP request to APNs that conforms to the following 

Requirements:
 * Set the value for the apns-push-type header field to liveactivity.
 * Set the apns-topic header field using the following format: <your bundleID>.push-type.liveactivity.
 * Set the value for the apns-priority header field to 5 or 10.

### Sample Payload for APNS to Update Activity

```swift

    /// APNS Sample body for updating live activity

    {
        "aps": {
          "timestamp": 1685952000,
          "event": "update",
          "content-state": {
            "currentHealthLevel": 0.0,
            "eventDescription": "Power Panda has been knocked down!"
        },
        "alert": {
                "title": "Power Panda is knocked down!",
                "body": "Use a potion to heal Power Panda!",
                "sound": "default"
                }
        }
    }


    /// Sample Curl Command to update live acitivty

    curl \
        --header "apns-topic: com.example.apple-samplecode.Emoji-Rangers.push-type.liveactivity" \
        --header "apns-push-type: liveactivity" \
        --header "apns-priority: 10" \
        --header "authorization: bearer $AUTHENTICATION_TOKEN" \
        --data '{
            "aps": {
            "timestamp": '$(date +%s)',
            "event": "update",
            "content-state": {
                        "currentHealthLevel": 0.941,
                        "eventDescription": "Power Panda found a sword!"
                    }
            }
 }' \
 --http2 https://api.sandbox.push.apple.com/3/device/$ACTIVITY_PUSH_TOKEN

```


## Determine the update frequency

The system allows for a certain budget of ActivityKit push notifications per hour. Set the HTTP header field apns-priority for your requests to specify the priority of an ActivityKit push notification:

* If you don’t specify the apns-priority value, APNs delivers the ActivityKit push notification immediately with the default priority of 10 and counts it toward the notification budget that the system imposes.

* If you exceed the budget, the system may throttle your ActivityKit push notifications.

> To avoid throttling, you can send a low-priority ActivityKit push notification that doesn’t count toward the budget by setting the HTTP header field apns-priority to 5. Consider this lower priority first before using the priority of 10. In many cases, choosing a mix of priority 5 and 10 for updates prevents your Live Activity updates from being throttled.

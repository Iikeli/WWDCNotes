# The Push Notifications primer

Help people get the most out of your app with push notifications for important events and updates — and by delivering up-to-date data in the background, so that it is ready when they open your app. Discover how you can use notifications and alert people to timely and relevant information. Learn the differences between alert and background notifications, how to adopt them in your apps, and avoid mistakes by using the right APIs for the job.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10095", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



## Types of Notifications

1. Alert notifications: Visible alerts, delivered in a way that allows customer to interact with it
2. Background notifications: Received while your app is not running, in order to keep the application content updated

## Alert Notifications

* Visible alerts
* Display new information
* Can be interactive
* Foreground or background app state
* Customizable

To implement the notifications:

1. Register for remote notifications in your `didFinishLaunchingWithOptions` using:

```swift
UIApplication.shared.registerForRemoteNotifications()
```

This allows the app to register for a remote notification and sends back a device token to be used to interact with the app through the user notification delegate

2. Implement `UNUserNotificationCenterDelegate` and assign the `AppDelegate` as the `UNUserNotificationCenter.current` delegate

Example of previous two steps implemented in `AppDelegate`:

```swift
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        UIApplication.shared.registerForRemoteNotifications()
        UNUserNotificationCenter.current().delegate = self
        return true
    }
	...
}
```

3. When registering for remote notifications, you’ll receive a callback on one of two delegate methods:

```swift
func application(_ application: UIApplication,
                   didFailToRegisterForRemoteNotificationsWithError error: Error) {
    // The token is not currently available.
    print("Remote notification is unavailable: \(error.localizedDescription)")
}

func application(_ application: UIApplication,
                   didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
     // Forward the token to your provider, using a custom method.
     self.forwardTokenToServer(token: deviceToken)
}
```

* If retrieving a device token has failed, you will receive a callback in `didFailToRegisterForRemoteNotificationsWithError` letting you know what went wrong. If it succeeds, you will receive the device token in `didRegisterForRemoteNotificationsWithDeviceToken`, which you will need to send your backend to be used later to send notifications to the app.

4. Forwarding device token can be done in a way similar to this:

```swift
func forwardTokenToServer(token: Data) {
    let tokenComponents = token.map { data in String(format: "%02.2hhx", data) }
    let deviceTokenString = tokenComponents.joined()
    let queryItems = [URLQueryItem(name: "deviceToken", value: deviceTokenString)]
    var urlComps = URLComponents(string: "www.example.com/register")!
    urlComps.queryItems = queryItems
    guard let url = urlComps.url else {
        return
    }

    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        // Handle data
    }

    task.resume()
}
```

5. Lastly, you need to ask for the user’s permission to send push notifications before doing so, this can be done using `UNUserNotificationCenter`:

```swift
let userNotificationCenter = UNUserNotificationCenter.current()
userNotificationCenter.requestAuthorization(options: [.alert, .sound, .badge]) { (granted, error) in
    print("Permission granted: \(granted)")
}
```

And that’s it. Now you just have to send the notifications from your backend to the previously saved device token. Here is a notification payload example of a restaurant app that wants to promote a daily special:

![][notification_example]

```js
{
    "aps" : {
       "alert" : {
            "title" : "Check out our new special!",
            "body" : "Avocado Bacon Burger on sale"
        },
        "sound" : "default",
        "badge" : 1,
   },
    "special" : "avocado_bacon_burger",
    "price" : "9.99"
}
```

* The `aps` dictionary is a standard Apple Push Notification Service object that contains information about how the notification should be rendered. Inside we have:
  * `alert` object, this tells the system what text to use inside the notification
  * `sound` is an optional field which should be included if you want the device to play a sound upon delivering the notification, other values can be provided for custom sounds
  * `badge` is also optional field that’s used to modify the badge of your app icon, this is an absolute value which will be displayed on the app icon, also this value can be modified by the app programmatically
* `special` and `price` are examples of custom data that you can provide to your application

## Parsing The Notification Payload

Here is how you can parse the previous sample notification payload:

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {
    let userInfo = response.notification.request.content.userInfo
    guard let specialName = userInfo["special"] as? String,
          let specialPriceString = userInfo["price"] as? String,
          let specialPrice = Float(specialPriceString) else {
        // Always call the completion handler when done.
        completionHandler()
        return
    }

    let item = Item(name: specialName, price: specialPrice)
		addItemToCart(item)
  	showCartViewController()
    completionHandler()
 }
```

> It is important to call `completionHandler` before returning from the function.

## Background Notifications

* Fetch data in background
* Keep application up to date
* Will launch the application if needed
* Managed by the system

To implement Background Notifications: 

- you just have to follow the same steps 1, 3, and 4 from Alert Notifications steps. 
- implementing the delegate in step 2 isn’t required because there won’t be any user response to those notifications
- Background Notifications don’t require asking for user’s permission as in step 5 of Alert Notifications.

An example of a Background Notification payload is this:

```js
{
    "aps" : {
       "content-available" : 1
    },
    "myCustomKey" : "myCustomData"
}
```

It’s a much simpler version of Alert Notifications payload, where you only have to provide the `content-available: 1` key-value pairs inside the `aps` dictionary, which tells the system that this is a Background Notification.   
As in Alert Notification, you can also send custom data inside your payload.

Handling a Background Notification is done by implementing the application delegate function `application(_ :didReceiveRemoteNotification:fetchCompletionHandler`:

```swift
func application(_ application: UIApplication,
                     didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                     fetchCompletionHandler completionHandler:
                     @escaping (UIBackgroundFetchResult) -> Void) {
    guard let url = URL(string: "www.example.com/todays-menu") else {
        completionHandler(.failed)
        return
    }

    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data else {
            completionHandler(.noData)
            return
        }
  
        updateMenu(withData: data)
        completionHandler(.newData)
    }
}
```

This function also requires calling the `completionHandler`, but you will have sometime before the application is terminated by the system. Call the `complettionHandler` with the result of your background process. This helps the system determines when it’s the best time to delivery your next Background Notifications.

Here are sample apps that shows implementing [Alert][AlertSampleApp] and [Background][BackgroundSampleApp] notifications.

[notification_example]: notification_example.png
[AlertSampleApp]: https://developer.apple.com/documentation/usernotifications/implementing_alert_push_notifications
[BackgroundSampleApp]: https://developer.apple.com/documentation/usernotifications/implementing_background_push_notifications

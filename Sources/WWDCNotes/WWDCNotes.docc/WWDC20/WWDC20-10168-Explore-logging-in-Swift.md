# Explore logging in Swift

Meet the latest generation of Swift unified logging APIs. Learn how to log events and errors in your app while preserving privacy. Take advantage of powerful yet readable options for formatting data — all without sacrificing performance. And we’ll show you how you can gather and process log messages to help you understand and debug unexpected behavior in your apps.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10168", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(ATahhan)
   }
}



## New Logging APIs

* Record events as they happen
* Archived on device for later retrieval
* Low performance overhead

### Adding Logs Steps:

1. Import the `os` framework

```swift
import os
```

2. Define a [logger][loggerDoc] object with: 
  - a `subsystem` identifier, which is used to make it clear that a message comes from our app.
  - a `category`, which differentiates messages coming from different parts of our app.

```swift
let logger = Logger(subsystem: "com.example.Fruta", category: "giftcards")
```

3. Add the logging action

```swift
logger.log("Started a task")
```

`log` behaves similar to a `print` statement, but it doesn't convert logs into `String`s like `print` does, it optimizes the printed logs based on the type of logged data.

We can log anything that is:

* A numeric type like `Double` and `Int`
* Objective-C objects with `-description`
* Any type that conforms to [`CustomStringConvertible`][customProtocolDoc]

Nonnumeric types are redacted from the log output, to make sure that no personal information are mistakenly logged. To explicitly mark those types as safe to log, we can pass the optional parameter `privacy` as such:

```swift
logger.log("Ordered smoothie \(smoothieName, privacy: .public)")
```

## Retrieving Logs

We can retrieve the logs by plugging the device into our mac and running the command:

```
log collect --device --start '2020-06-22 9:41:00' --output fruta.logarchive
```

> Logs are streamlined into the `Console.app` when our device is connected to our mac

## Log Levels

Those are the available logging levels sorted in an increasing order of their importance:

| Log Level | Description | Persistence |
| ----------- | ----------- | ----------- |
| Debug | Useful only during debugging | Not persisted |
| Info | Helpful but not essential for troubleshooting | Persisted only during `log` collect |
| Notice (default) | Essential for troubleshooting | Persisted up to a storage limit |
| Error | Error seen during execution | Persisted up to a storage limit |
| Fault | Bug in program | Persisted up to a storage limit |

* Not persisted or deleted logs can't be accessed later on. We can access not persisted logs only while streamlining logs directly from the device

* As the log level increases, the performance of that logging decreases: `Debug` is the most performant log level, while `Fault` is the least performant

* Because of the previous point, logging very slow functions at the `Debug` level is safe because the compiler will make sure that those aren't executed when the debug messages are discarded

## Formatting Logs

`Logger` offers many APIs that we can use to format our logs, for example:

```swift
statisticsLogger.log(
    """
    \(taskID) 
    \(giftCardID, align: .left(columns: GiftCard.maxIDLength)) 
    \(serverID) 
    \(seconds, format: .fixed(precision: 2))
    """
)
```

* Formatting data has no cost at run-time
* There are many other [formatting APIs][formattingDoc] and [alignment options][alignmentDoc]
* It's very important to remember that we shouldn't log any private information
* We can use the `hash` formatting option as a parameter to the privacy level of our logs to detect when two logged values are the same, without revealing the actual value:

```swift
logger.log("Paid with bank account: \(accountNumber, privacy: .private(mask: .hash))")
```

Which produces an output similar to:

```
Paid with bank account <mask.hash:'Csf11hACSI24GHca4sf8178er=='>
```

## API Availability

iOS 14, tvOS 14, watchOS 7, and macOS Big Sur:

* New Logger APIs
* New `os_log()` overloads that accept string interpolations

Prior releases:

* `os_log()` overloads that accepted only formatted static strings

[customProtocolDoc]: https://developer.apple.com/documentation/swift/customstringconvertible
[formattingDoc]: https://developer.apple.com/documentation/os/oslogfloatformatting
[alignmentDoc]: https://developer.apple.com/documentation/os/oslogstringalignment
[loggerDoc]: https://developer.apple.com/documentation/os/logger

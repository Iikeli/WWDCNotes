# Efficient Design with XPC

XPC has been enhanced to make it even easier to design for robustness and efficiency. Learn how to save power by opportunistically scheduling long-running tasks, transferring large amounts of data with minimal overhead, and how to best compartmentalize your app.

@Metadata {
   @TitleHeading("WWDC13")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc13/702", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What is [XPC][XPC]?

- It's a library that combines service bootstrapping and IPC (interprocess communication), a.k.a. everything related to having a service up and running and exchanging messages with it
- Helps refactoring an app into services (with different responsibilities, privileges, etc)
- These services are deployed within the app bundle

## Key Benefits

- fault isolation: if a service crashes the main app still runs fine
- different privileges/entitlements: even if your app has access to iCloud or the contacts library etc, this doesn't mean that your app services automatically inherits those. The app decides which privileges to grant to the service, use the principle of least required privilege
- XPC manages the lifecycle for all these services (no need to spawn/pause/etc)

## XPC Kinds

Two kinds, **Bundled Services** and **launchd Services**.

### Bundled Services

- Ship within an app bundle
- Stateless: meant to be stateless, on-demand helpers that come up to do something (a service, some requests)
- Fully managed lifecycle

### launchd Services

- Run as root
- independent from any app
- cannot distribute on the app store

In order to use these `launchd` services you must have a launchd plist in either [Library LaunchDaemons or Library LaunchAgents][demonsDoc].

## API

From high level to low level:

- [NSXPCConnection][NSXPCConnection]
- libxpc
- libdispatch
- libobjc

## Best practices

- Avoid long-running processes: the system prefers to launch them on on-demand and exit when they're not needed
- Adapt to resource availability
- Lazy initialization: don't do work unless the user has done something where you need to initialize your resources

## XPC Events

With XPC Events the system acts as the source of demands that trigger your service. This is done via [`launchd`][demonsDoc].

A few examples:

- [IOKit][IOKit] events: your service will be triggered (by `launchd`) whenever changes in the IO Registry happen
- [BSD notifications - Notify(3)][BSD]: you can post a notification and a `launchd` job triggers
- [`CFDistributedNotifications`][CFDistributedNotifications]: a newer alternative to BSD notifications

### Register for XPC events

In order to use XPC events you need to define which events can trigger your service via the `launchd.plist`, for example:

```plist
<key>LaunchEvents</key>
<dict>
  <key>com.apple.iokit.matching</key>
  <dict>
    <key>com.mycompany.device-attach</key>
    <dict>
	  <key>idProduct</key>
	  <integer>2794</integer>
	  <key>idVendor</key>
	  <integer>725</integer>
	  <key>IOProviderClass</key>
	  <string>IOUSBDevice</string>
	  <key>IOMatchLaunchStream</key>
	  <true/>
	</dict>
  </dict>
</dict>
```

### Consume XPC events

When these events are posted, your app need to consume them, for example:

```objc
xpc_set_event_stream_handler(“com.apple.iokit.matching”, q, ^(xpc_object_t event) {
	// Every event has the key XPC_EVENT_KEY_NAME set to a string that
	// is the name you gave the event in your launchd.plist.
	const char *name = xpc_dictionary_get_string(event, XPC_EVENT_KEY_NAME);

	// IOKit events have the IORegistryEntryNumber as a payload.
	uint64_t id = xpc_dictionary_get_uint64(event, “IOMatchLaunchServiceID”);

  // Reconstruct the node you were interested in here using the IOKit
	// APIs. 
});
```

This [`xpc_set_event_stream_handler`][xpc_set_event_stream_handler] takes three arguments:

- the first it the event identifier, to declare that this is the handler for IOKit Events for example
- the second is a dispatch queue
- the third is a block

The block gets invoked on that queue: once this block is consumed, the event is considered consumed.
Each notification has a payload that allows you to reconstruct who triggered along with other information (depending on the event).


## Centralized Task Scheduling

Based on XPC activity APIs, will help you schedule tasks at the right time (e.g. when the system is idle, along with other tasks) to minimize disruption to user experience.

Activity types:

- Maintenance (launched when the machine is in idle, interrupted when the user begins using the machine)
- Utility (interrupted when resources become scarce)

Activity Criteria:

- A/C power
- Battery level
- HDD spinning
- Screen asleep

Example of activity:

```objc
xpc_object_t criteria = xpc_dictionary_create(NULL, NULL, 0);
xpc_dictionary_set_int64(criteria, XPC_ACTIVITY_INTERVAL, 5 * 60); 
xpc_dictionary_set_int64(criteria, XPC_ACTIVITY_GRACE_PERIOD, 10 * 60);

// Activity handler runs on background queue. 
xpc_activity_register(“com.mycompany.myapp.myactivity”, criteria, ^(xpc_activity_t activity) {
	id data = createDataFromPeriodicRefresh();
	// Continue the activity asynchronously to update the UI. 
	xpc_activity_set_state(activity, XPC_ACTIVITY_STATE_CONTINUE); 
	dispatch_async(dispatch_get_main_queue(), ^{
	    updateViewWithData(data);
		xpc_activity_set_state(activity, XPC_ACTIVITY_STATE_DONE); 
	});
});
```

## Service Lifecycle

- Service launches on-demand 
- System stops service as needed
  - App quits
  - Memory pressure
  - Idle/lack of use

## Importance Boosting

By default processes are launched in a background queue, however sometimes we need the service to process something immediately (to avoid bad user experience): use importance boosting for such scenarios. Importance boosting makes sure that the service gets all the resources etc.

Use the `ProcessType` key in the `launchd.plist` to opt into this behavior, possible values:

| value | Contention Behavior | Use when |
| Adaptive | contends with apps when doing work on their behalf | app uses XPC to communicate with launchd job |
| Background | Never contend with apps | app has no dependency on launchd job’s work |
| Interactive | Always contend with apps | Extreme cases (Apple doesn't want you to use this) |
| Standard | Default behavior |  |

## Debugging Tips

- use `imptrace(1)` tool for debugging important boost services

- If you get a `connection-invalid` error, this indicates a configuration error: 
  - make sure service target is dependency of app target
  - make sure service target is in Copy Files build phase
  - make sure `CFBundleIdentifier` matches service name

- When your service "misbehaves" (obvious misuse of certain APIs etc), it can be killed:
  - from clients you will get a crash report
  - during debugging, you can use [`xpc_debugger_api_misuse_info()`][xpc_debugger_api_misuse_info] in lldb to get a pointer to the human-readable string describing the reason the caller was aborted.

Crash report example:

```
Exception Type: EXC_BAD_INSTRUCTION (SIGILL)
Exception Codes: 0x0000000000000001, 0x0000000000000000
Application Specific Information:
API MISUSE: Over-release of an object
```

lldb example:

```lldb
Exception Type: EXC_BAD_INSTRUCTION (SIGILL)
Exception Codes: 0x0000000000000001, 0x0000000000000000
 Application Specific Information:
API MISUSE: Over-release of an object
```

[xpc_debugger_api_misuse_info]: https://developer.apple.com/documentation/xpc/1505415-xpc_debugger_api_misuse_info?language=objc
[xpc_set_event_stream_handler]: https://developer.apple.com/documentation/xpc/1505578-xpc_set_event_stream_handler?language=objc
[CFDistributedNotifications]: https://developer.apple.com/documentation/corefoundation/cfnotificationcenter?language=objc
[BSD]: https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man3/notify.3.html
[IOKit]: https://developer.apple.com/documentation/iokit
[NSXPCConnection]: https://developer.apple.com/documentation/foundation/nsxpcconnection
[demonsDoc]: https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html
[XPC]: https://developer.apple.com/documentation/xpc
# Measuring Performance Using Logging

Learn how to use signposts and logging to measure performance.  Understand how the Points of Interest instrument can be used to examine logged data. Get an introduction into creating and using custom instruments.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/405", purpose: link, label: "Watch Video (35 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



Signposts - a new feature of the [`os_log`](https://developer.apple.com/documentation/os/logging) API

- useful for debugging performance issues
- integrated with Instruments, which can visualize activity over time using signposts

Signposts allow you to mark the beginning and end of a piece of work and mark it with some kind of label:

```swift
import os.signpost

let refreshLog = OSLog(subsystem: "…", category: "…")

os_signpost(.begin, log: refreshLog, name: "Fetch Asset")
// …do actual work…
os_signpost(.end, log: refreshLog, name: "Fetch Asset")
```

Ranges of signposts with different names can overlap - e.g. you can have one signpost covering the whole process and smaller signposts covering specific single tasks that it consists of:

```swift
os_signpost(.begin, log: log, name: "Load Data")

os_signpost(.begin, log: log, name: "Fetch Asset")
// …
os_signpost(.end, log: log, name: "Fetch Asset")

os_signpost(.begin, log: log, name: "Parse JSON")
// …
os_signpost(.end, log: log, name: "Parse JSON")

os_signpost(.end, log: log, name: "Load Data")
```

If you run multiple tasks of the same kind, to let the system differentiate between them and know which begin matches which end, you can add a “signpost ID”:

```swift
os_signpost(.begin, log: log, name: "Load Data")

for asset in assets {
    let spid = OSSignpostID(log: log)

    os_signpost(.begin, log: refreshLog, name: "Fetch Asset", signpostID: spid)
    // …do actual work…
    os_signpost(.end, log: refreshLog, name: "Fetch Asset", signpostID: spid)
}

os_signpost(.end, log: log, name: "Load Data")
```

You can also pass your model object when generating the signpost ID - then it will always use the same signpost ID for the same object, and you don't have to store the signpost object, just your model:

```swift
let spid = OSSignpostID(log: log, object: asset)
```

There is also an option of passing an additional string argument to begin/end to provide some context (e.g. to differentiate between different possible ways to finish an activity, like success/failure). The string also accepts format arguments like `os_log`:

```swift
os_signpost(.begin, log: log, name: "Compute Physics",
    "Calculating %{public}s: %d %d %d %d", description, x1, y2, x2, y2)
```

Apart from marking the beginning and end, you can also mark specific points in time during the process using the `.event` signpost type:

```swift
os_signpost(.event, log: log, name: "Fetch Asset",
    "Received chunk of data, size %d", size)
```

Signposts are very optimized internally, they’re built to minimize the time spent when logging, same as the whole `os_log` API. This means you can emit a lot of signposts even in a very tight window when investigating a performance bottleneck.

If you still really want to enable or disable some signpost logs based on some conditions, you can swap your logger object with [`OSLog.disabled`](https://developer.apple.com/documentation/os/oslog/2863695-disabled):

```swift
let refreshLog: OSLog

if ProcessInfo.processInfo.environment.keys.contains("SIGNPOSTS_REFRESH") {
    refreshLog = OSLog(…)
} else {
    refreshLog = .disabled
}
```

To conditionally disable some expensive code that is only useful for debugging, you can check the [`signpostsEnabled`](https://developer.apple.com/documentation/os/oslog/3006881-signpostsenabled) property:

```swift
if refreshLog.signpostsEnabled {
    let information = collectInfo()
    os_signpost(…, information)
}
```

All APIs are also available in C & ObjC:

```objc
#include <os/signpost.h>

os_signpost_interval_begin()
os_signpost_interval_end()
os_signpost_event_emit()
os_signpost_id_t
OS_LOG_DISABLED
```

Use the formatter `%{xcode:size-in-bytes}u` to let Xcode & Instruments know that the logged value is a byte size of some data

- This is one of so called “Engineering types” - find more in the Instruments Help menu, in the Instruments developer guide


## Instruments tips

- Use the “os_signpost” instrument to profile using signposts
- After recording some data, you can see the signpost names in the sidebar on the left and signpost ranges with the optional begin/end comments marked on the chart
- In the bottom pane you can see count and duration statistics, grouped by category, signpost, id and comment
- Clicking the arrow button next to a specific message row shows you a list of all instances of this specific message (selecting them highlights them on the timeline)
- For metadata like logged byte sizes of downloads, you can choose “Summary: Metadata Statistics” to see total/min/max/avg of each type of value
- Live streaming signpost logs to Instruments (“Immediate mode”) adds some overhead, so if you want to avoid that while debugging some performance-critical code, click and hold the Record button to access recording options and change mode to “Last n seconds”


## Points of interest

Points of interest is a special log category of `OSLog`. It’s meant for logging important actions taken by the user, like opening some specific screen.

Normal `os_log` logs and signpost logs logged to this category appear in a special separate Instruments timeline, which lets you visualize what was happening in the app at the moment when something happened on other charts like CPU usage.


## Custom Instruments packages

You can now build your custom Instruments packages, defined as an XML file in a separate target, which appear as a new kind of template when starting Instruments. This lets you process and present collected signpost data in a different way that makes sense for the specific problem you’re analyzing.

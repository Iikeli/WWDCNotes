# Advances in App Background Execution

Background execution is a powerful tool your app can leverage to provide a great user experience. Learn about best practices to follow when running in the background, especially if you use VoIP or silent pushes, and an all-new scheduling API that enables long running processing and maintenance tasks.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/707", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Background Execution Best Practices 

If we want something to happen now or almost now, even when the app goes to the background, we need to call:

- from main app: [`UIApplication.beginBackgroundTask(expirationHandler:)`][beginBackgroundDoc]
- from extensions: [`ProcessInfo.performExpiringActivity(withReason:using:)`][performExpiringDoc]

These objects give us more run time in the case our app is not in the foreground anymore. This api will ensure that our request goes through before the app gets suspended.

## Discretionary Background URL Session 

- Defer the download to later, when is “later” is decided by the system (could be hours!). 
- This is for downloading content that is not high priority (or batch analytics) and to do so when is more proper.
- Allows the system to defer the download until a better time
- Provide information to system for smarter scheduling 

```swift
// Set up background URL session 
let config = URLSessionConfiguration.background(withIdentifier: "com.app.attachments")
let session = URLSession (configuration: config, delegate: ..., delegateQueue: ...)

// Set discretionary
config.discretionary = true 
```

We can even set a the earliest time when to start a download and an estimate workload size so the system can make some assumption on when it is the right time to do so.

Note that, if we put it too far in the future, and the user never uses the app again, our task might never be executed.

```swift
// Set time window
task.earliestBeginDate = Date(timeIntervalSinceNow: 2 * 60 * 60)

// Set workload size 
task.countOfBytesClientExpectsToSend = 160
task.countOfBytesClientExpectsToReceive = 4096
task.resume() 
```

## [`BackgroundTasks` Framework][btdocs]

- New in iOS 13
- Example of tasks that fit perfectly with this framework:
  - Data Syncing 
  - Database Cleanup
  - Backups Upload 

### Background Processing Tasks 

Features:

- Several minutes of runtime at system-friendly times 
- Option to turn off CPU Monitor for intensive work: 
- at night when the device is charging we can do battery intensive work without worrying 

### Background App Refresh Task 

- 30 seconds of runtime to update the app every time
- Called based on the user usage pattern: our app will get this refresh task before the user launches the app (again, based on the pattern) so that when the user launches the app all the new content has been downloaded already.
- See [`BGAppRefreshTask`][bgapprefreshtaskDocs]

## How to use `BackgroundTasks`

We will interact mainly via the [`BGTaskScheduler`][schedulerDocs], which is the interface to all of this intelligent scheduling.

To create a request, we will create an instance of either `BGProcessingTaskRequest` or `BGAppRefreshTaskRequest`, based on the task.

When the system is ready to give us time to do the task, the app will be launched (in the background) with a `BGAppRefreshTask` / `BGAppProcessingTask`.

Once we’re done, we will call [`setTaskCompleted`][completed] and let the app be suspended.

Based on the tasks we’ve given to the `BGTaskScheduler`, our app will be given multiple tasks at the same time. However note that the time allotted for this is the same if it was one or multiple tasks.

All the tasks will be given to the main app, even when the background task was scheduled by an extension

## HOW-TO

1. We need to have the background mode capability turned on with background fetch/processing ticked.

2. Set the `info.plist` keys to tell iOS we support app background task/refresh. Add a key “Permitted background task scheduler identifiers”  and use an array of strings as the value. This array contains unique strings that our apps must declare, each string declares a different task that the app wants to perform.

3. Go in the app delegate and deal with the task:
  - import BackgroundTasks
  - Register a background task.

```swift
BGTaskScheduler.shared.register(
	forTaskWithIdentifier: "com.colorfeed.refresh", 
	using: nil) { task in 
  self.handleAppRefresh(task: task as! BGAppRefreshTask)
}
```

Note how the last block of this function is what will be called when the system decides that it’s time to do the task.
Remember to call `task.setTaskCompleted` once it is done.

4. Lastly, schedule the task

```swift
func scheduleAppRefresh() {
	let request =BGAppRefreshTaskRequest(identifier: "com.colorfeed.refresh")
	do { 
	 	try BGTaskScheduler.shared.submit(request) 
	} catch { 
		print("Could not schedule app refresh: \(error)") 
	}
}
```

Some cool properties of these requests:

- `earliestBeginDate` tells the system to wait at least the time we pass to this method before calling the task

- `requiresNetworkActivity` (for processing only) tells the system to wake up our app for the task only if we have connectivity (or if we don’t need connectivity to do our task)

- `requiresExternalPower` tells the system we’re going to do intensive stuff and use lots of resources. Setting this to `true` also disables CPU Monitor, allowing the app to do intensive work with no throttling.


## Debugging

After scheduling the task, we can mock/force a system call to activate our scheduled task via the debugger with:

```
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"TASK_IDENTIFIER"]
```

(replace `TASK_IDENTIFIER` with the task name.

[beginBackgroundDoc]: https://developer.apple.com/documentation/uikit/uiapplication/1623031-beginbackgroundtask
[performExpiringDoc]: https://developer.apple.com/documentation/foundation/processinfo/1617030-performexpiringactivity
[btdocs]: https://developer.apple.com/documentation/backgroundtasks
[bgapprefreshtaskDocs]: https://developer.apple.com/documentation/backgroundtasks/bgapprefreshtask
[schedulerDocs]: https://developer.apple.com/documentation/backgroundtasks/bgtaskscheduler
[completed]: https://developer.apple.com/documentation/backgroundtasks/bgtask/3142236-settaskcompleted
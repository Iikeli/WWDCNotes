# Debug with structured logging

Discover the debug console in Xcode 15 and learn how you can improve your diagnostic experience through logging. Explore how you can navigate your logs easily and efficiently using advanced filtering and improved visualization. We’ll also show you how to use the dwim-print command to evaluate expressions in your code while debugging.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10226", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(rogerluan)
   }
}



## What's new in the debug console

Now you can toggle metadata associated with the console output, making it a lot easier to visualize things urgency, category, and origin of the log message:

![Console metadata][console-metadata]

If you don't want to enable the metadata for all logs, you can quickly view the metadata of a single log line by pressing space on it:

![Quick look][quick-look]

Filtering received great improvements as well, so your text-based searches can filter through the multiple types of metadata that Xcode collects, as well as toggle the visibility of different log types:

![Filtering][filtering]

And you can also quickly show or hide similar items by right-clicking on a log:

![Toggle similar items][toggle-similar-items]

By hovering the mouse over a log, you can also see the full path of the file that generated the log, and clicking on it will open the file in the editor.

## LLDB improvements

Most of us simply use `po` whenever we want to debug anything in the console. This is often not the right command to use, as it can be a lot slower to perform than other simpler commands, and might not give what you want, e.g. by simply printing the memory address of the object, which's not helpful at all. There are other alternatives:

![LLDB commands][lldb-commands]

However, Apple has added a new command that can be used at all times and will generally do what you want, printing the best description possible of what you're trying to print. It's called Do What I Mean Print — `dwim-print`.

Since `dwim-print` is quite long and we don't want to be typing it all the time, they replaced the `p` and `po` aliases with this new smarter print command, so you can generally simply always just use `p` from now on 🎉

## Tips for logging

Best practices reminder:

- `stdio` is for command-line UI
- `OSLog` is for debugging

Therefore, `print(…)` should rarely be used to log events in your program's execution. Thus, move away from this:

![Logging bad practice][logging-bad-practice]

To this:

![Logging best practice][logging-best-practice]

Finally, to get most out of your logging:

- Create multiple log handles, specific to each area of your application, to help you find the logs you're looking for.
- Take advantage of `OSLogStore` to collect valuable diagnostics when issues occur with your application in production.
- `OSLog` is a tracing facility. This means it is capable of providing you with complex performance analysis of your application using tools like Instruments.

[console-metadata]: WWDC23-10226-console-metadata
[quick-look]: WWDC23-10226-quick-look
[filtering]: WWDC23-10226-filtering
[toggle-similar-items]: WWDC23-10226-toggle-similar-items
[lldb-commands]: WWDC23-10226-lldb-commands
[logging-bad-practice]: WWDC23-10226-logging-bad-practice
[logging-best-practice]: WWDC23-10226-logging-best-practice

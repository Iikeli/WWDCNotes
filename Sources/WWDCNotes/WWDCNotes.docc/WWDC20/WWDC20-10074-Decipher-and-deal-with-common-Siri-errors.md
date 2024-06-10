# Decipher and deal with common Siri errors

“Sorry, there was a problem with the app..."

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10074", purpose: link, label: "Watch Video (2 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Automate Siri Queries

- You can automate Siri queries using the scheme editor in Xcode when debugging your extension
- When you're attaching to your intents extension, you have an option to choose between Siri and the Shortcuts.app as the host process

## Debugging intents

- as the intents UI extensions run in a separate processes, your breakpoints won't be hit by default: use the Xcode debug menu to attach to multiple processes at the same time

## "Sorry there was a problem with the app"

If you get this error:

- make sure your intent handler calls the completion handler within 10 seconds
- make sure the completion handler is called once
- your app/extension might have crashed (check crash logs)

## Tips

- use `os_log` and Console.app to see how your app and its extensions interact with each other
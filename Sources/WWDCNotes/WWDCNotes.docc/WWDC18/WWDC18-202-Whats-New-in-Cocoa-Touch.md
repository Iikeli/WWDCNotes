# What's New in Cocoa Touch

iOS 12 enhances the Cocoa Touch frameworks to improve app performance and deliver exciting new features. Learn about performance best practices, security improvements, tools for supporting multiple screen sizes and shapes, new APIs for iMessage apps, Siri Shortcuts, and Swift refinements. Find out which sessions you won't want to miss throughout the week.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/202", purpose: link, label: "Watch Video (40 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Challenge

`cellForRow` has:

- `16ms` to execute in a `60hz` screen device
- `8ms` in a 120hz screen device 

If the function takes longer than that, the app fps (and user experience) will drop. 

## Solution

From iOS 10 we have a prefetch data function (that runs in the background) which helps us fetching the data before it being required to be displayed in a cell:

```swift
// UITableView Pre-Fetching

protocol UITableViewDataSourcePrefetching { 
  func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath])
  func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt: indexPaths [IndexPath])
}
```

## Improvements Under The Hood

Beside the prefetching above, this session goes deep into a lot of system improvements (cpu, different threads management, auto layout).
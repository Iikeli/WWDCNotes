# Modernizing Your UI for iOS 13

iOS 13 combines powerful new multitasking and productivity technologies with a refreshed look and feel for all applications. Familiarize yourself with new requirements to keep your app ready for upcoming versions of iOS. Discover improvements to search fields and a totally new way to present UI. Learn how to adopt new gestures around selection, access peek and pop on all hardware, and how to renew your app's look for iOS 13.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/224", purpose: link, label: "Watch Video (50 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- **Deprecation of LaunchImages** use Launch Storyboards instead
- **New App Store Requirement** Any app linking to iOS 13 must ensure correct layout at any size 
- **Split Screen Multitasking** is strongly recommended for iPad Apps
  - Apple is going to expect that **most** of the applications, **unless** you want to provide a truly immersive experience, are going to support split screen multitasking

- **UINavigationBarAppearance** customize `.standardAppearance` (iPhone portrait), `.compactAppearance` (iPhone landscape), `.scrollEdgeAppearance` (if attached to scrollView and scrolled to top, bar uses this one)
  - settable per `navigationItem` so customization per view controller possible (including color, transparency, …)

```swift
let appearance = navigationBar.standardAppearance.copy()
/* configuration */
navigationItem.standardAppearance = appearance
```

- **UIBarButtonAppearance, UIToolBarAppearance**
- **UITabBarAppearance** customize `.stackedLayoutAppearance` (text below icon), `.inlineLayoutAppearance` (text right to icon; iPads), `.compactInlineLayoutAppearance` (text right to icon; landscape; smaller phones)
- **UIModalPresentationStyle.[automatic | formSheet | pageSheet]** are able to stack behind each other
  - `.automatic` is the new default; gets resolved at presentation time
  - get the old behavior by `viewController.modalPresentationStyle = .fullScreen`
  - sheets get gesture recognizer added automatically to support `pull-to-dismiss` - opt-out possible (see link in description for example project)
  - **Appearance Callbacks Not Called for .PageSheet and .FormSheet** on presentingViewController:
    `viewWillAppear`, `viewDisappear`, `viewWillDisappear`, `viewDidDisappear`

- **Search UI** 
  - offers possibility to hide cancel button and scope control
  - expose `UISearchTextField` on `UISearchBar` so customization becomes possible
  - `Search Tokens` that represent complex searches
    - are copy&pastable and support drag&drop
    - can be created by user/developer from selected text in UISearchTextField

- **UITextInteraction** the new & easy way to control selection UI
- **Multi Selection in Table-/Collection Views** simply by Two-Finger-Swipe 
  - opt-in by implementing only 1 new delegate method for Tables and Collections

- **New Editing Gestures**
  - `pinch 3 fingers in`: copy selected text
  - `pinch 3 fingers out` paste copied text
  - `3 finger tap || 3 finger swipe back` undo
  - `3 finger swipe forward` redo
  - opt out by `responder.editingInteractionConfiguration = .none`

- **UIContextMenuInteraction** context menus with `rich preview`, `nested menus`, `in-line sections`
  - transform to context menus on macOS
  - `UIMenu` & `UIInteraction` provide hierarchical menu construction system
  - adopt by `view.addInteraction(UIContextMenuInteraction(delegate: self))`
  - customize how  view and menu will be presented: `UITargetedPreview & UITargetedDragPreview`
  - convenience delegate methods for context menus on UITableView & UICollectionView
  - `UIViewControllerPreviewing` (peek & pop) is deprecated for sake of this new API
  - replace long-press driven behavior/menus by this new API

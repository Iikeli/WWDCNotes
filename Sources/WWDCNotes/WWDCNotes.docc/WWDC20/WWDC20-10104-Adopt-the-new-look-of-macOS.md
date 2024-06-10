# Adopt the new look of macOS

Make over your Mac apps: Discover how you can embrace the new design of macOS Big Sur and adopt its visual hierarchy, design patterns, and behaviors. We’ll explore the latest updates to AppKit around structural items and common controls, and show you how you can adapt more customized interfaces with just a bit of adoption work. And find out how you can incorporate custom accent colors and symbols to further personalize your app.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10104", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(mackuba)
   }
}



macOS has an all new design:

- a new toolbar with inline title, big bold controls and integration with the window’s split view
- full height sidebars with colorful icons and updated symbol iconography
- new appearance for lists/tables using a new inset selection style

Most of these changes happen automatically to your app (if you’re using classes like [`NSToolbar`](https://developer.apple.com/documentation/appkit/nstoolbar) and [`NSSplitViewController`](https://developer.apple.com/documentation/appkit/nssplitviewcontroller)).


## Sidebars

Sidebar: use [`NSSplitViewController`](https://developer.apple.com/documentation/appkit/nssplitviewcontroller), items configured using [`NSSplitViewItem.Behavior.sidebar`](https://developer.apple.com/documentation/appkit/nssplitviewitem/behavior/sidebar)

- use full size content window mask - [`NSWindow.StyleMask.fullSizeContentView`](https://developer.apple.com/documentation/appkit/nswindow/stylemask/1644646-fullsizecontentview), so that content is laid out in the space normally taken by the title bar

[`NSView`](https://developer.apple.com/documentation/appkit/nsview) properties to find the safe areas of the view:

- [`safeAreaInsets`](https://developer.apple.com/documentation/appkit/nsview/3553227-safeareainsets), [`safeAreaLayoutGuide`](https://developer.apple.com/documentation/appkit/nsview/3553228-safearealayoutguide), [`safeAreaRect`](https://developer.apple.com/documentation/appkit/nsview/3553229-safearearect)
- also available on the storyboard (“Safe Area Layout Guide”)

New view item in the library: "Window Controller with Sidebar" ([`NSWindowController`](https://developer.apple.com/documentation/appkit/nswindowcontroller) + [`NSSplitViewController`](https://developer.apple.com/documentation/appkit/nssplitviewcontroller))

Opting out of the full height sidebar: [`NSSplitViewItem.allowsFullHeightLayout`](https://developer.apple.com/documentation/appkit/nssplitviewitem/3608197-allowsfullheightlayout)

- use this when sidebar is typically collapsed, or when you need more space for the toolbar

Accent colors:

- by default all sidebar icons are colored with user's chosen accent color
- use [`NSOutlineViewDelegate`](https://developer.apple.com/documentation/appkit/nsoutlineviewdelegate) method [`outlineView(_: tintConfigurationForItem:)`](https://developer.apple.com/documentation/appkit/nsoutlineviewdelegate/3626816-outlineview) to customize, return an instance of [`NSTintConfiguration`](https://developer.apple.com/documentation/appkit/nstintconfiguration):
  - [`.default`](https://developer.apple.com/documentation/appkit/nstintconfiguration/3626820-default) - always uses the system accent color
  - [`.monochrome`](https://developer.apple.com/documentation/appkit/nstintconfiguration/3626822-monochrome) - gray monochrome
  - [`init(preferredColor:)`](https://developer.apple.com/documentation/appkit/nstintconfiguration/3626824-init) - use this color when default (“rainbow”) accent color is used, but follow the accent color if it’s customized
  - [`init(fixedColor:)`](https://developer.apple.com/documentation/appkit/nstintconfiguration/3626823-init) - a fixed color that is always used (e.g. the yellow star in Mail’s VIP folder)
- use sidebar colors to distinguish different sections of the sidebar, or highlight a specific item like the VIP star
- or use monochrome to de-emphasize groups


## Toolbars

There’s no longer any special material behind the toolbar items, it’s a uniform part of the content of the window (this works automatically).

New toolbar styles, controlled through [`NSWindow.toolbarStyle`](https://developer.apple.com/documentation/appkit/nswindow/3608199-toolbarstyle) - [`NSWindowToolbarStyle`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle):

[`.unified`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle/unified)

- the new standard - like in the new versions of system apps
- larger controls, bold icons
- inline title located at the leading edge of the title bar next to the sidebar
- good choice for most windows

[`.unifiedCompact`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle/unifiedcompact)

- more compressed style
- regular sized controls, smaller toolbar height
- this is what was previously used if the window was configured to hide the title bar
- now supports an optional inline title
- use when user’s focus should be on the content of the window and there aren’t many elements in the toolbar

[`.preference`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle/preference)

- specifically designed for preference windows
- automatic when you’re using [`NSTabViewController`](https://developer.apple.com/documentation/appkit/nstabviewcontroller) with the [`.toolbar`](https://developer.apple.com/documentation/appkit/nstabviewcontroller/tabstyle/toolbar) tab style

[`.expanded`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle/expanded)

- what used to be the standard layout of the toolbar
- title is centered on top of the toolbar and can expand across the window
- large button icons with labels below
- use when the window title is long, or the toolbar is heavily populated with items, or when you want to keep existing toolbar layout

[`.automatic`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle/automatic)

- default value
- determines the toolbar style based on your window structure
- existing apps linked on older SDKs keep their old layout

Toolbar buttons no longer have a border, a shape only appears when hovering

- controls with text fields show a slight border so that you know where you can click to focus

[`NSToolbarItem`](https://developer.apple.com/documentation/appkit/nstoolbaritem) [`minSize`](https://developer.apple.com/documentation/appkit/nstoolbaritem/1531777-minsize) & [`maxSize`](https://developer.apple.com/documentation/appkit/nstoolbaritem/1526451-maxsize) properties are deprecated - macOS can automatically give your controls a proper size; you can still use constraints if necessary.

New [`NSWindow.subtitle`](https://developer.apple.com/documentation/appkit/nswindow/3608198-subtitle) property:

- shows a smaller subtitle under the window title, e.g. the message count in Mail
- in [`.expanded`](https://developer.apple.com/documentation/appkit/nswindow/toolbarstyle/expanded) style it appears next to the title

Controls like back/forward buttons should be put on the leading edge of the title bar, before the title

- set [`NSToolbarItem.isNavigational`](https://developer.apple.com/documentation/appkit/nstoolbaritem/3622481-isnavigational) to position them there
- users can add and remove them from the toolbar, but can only put them in the leading edge area

[`NSSearchToolbarItem`](https://developer.apple.com/documentation/appkit/nssearchtoolbaritem):

- new toolbar control for search text fields
- appears as a text field if there’s enough space, otherwise collapsed into an icon
- use [`searchField`](https://developer.apple.com/documentation/appkit/nssearchtoolbaritem/3634330-searchfield) property to access the text field itself
- works on older versions of macOS

[`NSTrackingSeparatorToolbarItem`](https://developer.apple.com/documentation/appkit/nstrackingseparatortoolbaritem):

- a separator that extends the separator line of the window content’s split view upwards through the toolbar
- when creating, you pass it the split view to track and the index of its divider

To position items in the title bar area of the sidebar, include an [`NSToolbarItem.Identifier.sidebarTrackingSeparator`](https://developer.apple.com/documentation/appkit/nstoolbaritem/identifier/3622482-sidebartrackingseparator) item and add those items *before* the sidebar separator item.

The toolbar has no border below, but a slight shadow appears below it to separate it from the content when the content is scrolled

- this happens if the scroll view fills the frame and you’re using [`.fullSizeContentView`](https://developer.apple.com/documentation/appkit/nswindow/stylemask/1644646-fullsizecontentview)
- otherwise, there will be a separator regardless of the scrolling position
- customize toolbar separator in the window using [`NSWindow.titlebarSeparatorStyle`](https://developer.apple.com/documentation/appkit/nswindow/3622489-titlebarseparatorstyle), or in split view per section using [`NSSplitViewItem.titlebarSeparatorStyle`](https://developer.apple.com/documentation/appkit/nssplitviewitem/3622473-titlebarseparatorstyle)


## Controls

New modern design of controls like popup buttons, sliders, segmented controls

New multicolor system accent color - uses each app’s preferred accent color

- define the app’s global accent color in the asset catalog (can be different for light/dark mode) + set the name in build options
- people can still chose one of the previously available accent colors, and then that color overrides your setting so they can use that color everywhere
- it’s preferred to use named colors like [`controlAccentColor`](https://developer.apple.com/documentation/appkit/nscolor/3000782-controlaccentcolor), [`selectedContentBackgroundColor`](https://developer.apple.com/documentation/appkit/nscolor/2998830-selectedcontentbackgroundcolor), [`keyboardFocusIndicatorColor`](https://developer.apple.com/documentation/appkit/nscolor/1532031-keyboardfocusindicatorcolor) instead of explicitly using your own color for controls

New [`.large`](https://developer.apple.com/documentation/appkit/nscontrol/controlsize/large) control size

- e.g. when you need one large action button
- works for: a few kinds of buttons, text fields, search fields, segmented controls
- also used in the unified toolbar style, and in system alerts

New inset style for table selection - adds extra padding, taller default row heights

[`NSTableView.style`](https://developer.apple.com/documentation/appkit/nstableview/style):

- [`.automatic`](https://developer.apple.com/documentation/appkit/nstableview/style/automatic) (default)
- [`.fullWidth`](https://developer.apple.com/documentation/appkit/nstableview/style/fullwidth) - edge to edge selection background, like previously
- [`.inset`](https://developer.apple.com/documentation/appkit/nstableview/style/inset)
- [`.sourceList`](https://developer.apple.com/documentation/appkit/nstableview/style/sourcelist) - new appearance of sidebar source list
- automatic style uses `.inset` by default (on apps built on the latest SDK), `.fullWidth` in bordered scroll views, `.sourceList` in source lists
  - you can check the [`effectiveStyle`](https://developer.apple.com/documentation/appkit/nstableview/3622474-effectivestyle) property to see what style is actually used
- the old [`SelectionHighlightStyle.sourceList`](https://developer.apple.com/documentation/appkit/nstableview/selectionhighlightstyle/sourcelist) setting is deprecated


## Text

System text styles are now available (Large Title, Headline, Body etc.) - but without Dynamic Type, so they have one constant size.

```swift
NSFont.preferredFont(forTextStyle: options:)
NSFontDescriptor.preferredFontDescriptor(forTextStyle: options:)
```

## Symbol images

SF Symbols is now available on the Mac:

```swift
NSImage.init?(systemSymbolName: accessibilityDescription:)
```

- they can scale to different font sizes and weights
- toolbar and sidebar items automatically configure symbol images to match the size & style of the container
- it’s best to use them inside [`NSImageView`](https://developer.apple.com/documentation/appkit/nsimageview) (see [`symbolConfiguration`](https://developer.apple.com/documentation/appkit/nsimageview/3667456-symbolconfiguration) property)
- to customize symbol configuration, use [`NSImage.withSymbolConfiguration(…)`](https://developer.apple.com/documentation/appkit/nsimage/3656508-withsymbolconfiguration)
- most of existing named system images now return some kind of symbol image from SF Symbols

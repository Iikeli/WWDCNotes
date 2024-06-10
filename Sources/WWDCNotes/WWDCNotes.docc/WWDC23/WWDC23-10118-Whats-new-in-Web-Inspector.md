# What’s new in Web Inspector

Web Inspector provides a powerful set of tools to debug and inspect web pages, web extensions, and WKWebViews on macOS, iOS and iPadOS. We’ll share the latest updates, including improved typography inspection, editing tools for variable fonts, controls to emulate people’s preferences, element badges in the DOM node tree, and Symbolic breakpoints.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10118", purpose: link, label: "Watch Video (28 min)")

   @Contributors {
      @GitHubUser(RamitSharma991)
   }
}



Web Inspector is part of Safari on macOS, and it provides a powerful set of tools that allow you to inspect all the resources and activities of a website, web app, web extension, or home screen web app.Many improvements have been made to developer features in Safari, including a streamlined way to pair and inspect web content on devices, a quick way to open a web page in a Simulator, a refreshed Responsive Design Mode, and many others. To enable Web inspector: 
- Open the Settings window of Safari and switch to the Advanced tab. 
- At the bottom, you'll find the setting to "Show features for web developers.
- Click the checkbox to enable this. The Develop menu is now available in the menu bar.
- Open Web Inspector using the menu item called Show Web Inspector or by pressing the keyboard shortcut Option+Command+I on any web page.

## Typography inspection
Web Inspector can help with a host of typography inspection tools. You can find them in the Details sidebar of the Elements tab, in the Font panel. 
 - It shows properties and capabilities of the font used on the selected element, such as the name of the primary font used. 
 - If the selected node includes characters for which the primary font does not have glyphs, a fallback font will be used for those.
 - Font panel shows a summary of the basic font properties, like font size, style, weight, and stretch.
 - There's also a section that shows the supported font feature properties and their used values. 
 - These toggle special aspects of a typeface such as ligatures, small caps instead of lowercase characters, special numeric styles, and many others.
 - Now the Font panel shows warnings for synthetic bold or oblique styles. 
 - Often, these aren't slanted versions of the upright style, but instead they're cursive, specially designed to convey a particular aesthetic.
 - Similar happens for synthetic bold, where the stroke of the glyphs is made artificially thicker. 
 - Web Inspector now displays a warning when a synthetic bold or oblique is used. 
 - It shows up next to the synthesized weight or style in the basic properties section of the Font panel. 
 - This warning can be a hint that the expected font file was not loaded. 
 - Variable fonts can help when the font file just doesn't support the exact values you asked for. 
 - All aspects of a variable font that can be configured are expressed as variation axes. 
 - Web Inspector also helps inspect an element that uses a variable font, the Font panel shows you a list of the supported variation axes. 
 - For each, it shows its `axis tag--` a `four-character identifier--` an `optional axis label`, the supported minimum and maximum values, as well as the current value, or the default value if one is not specified in CSS.
 - Interactive controls to edit the values of variation axes and see the results live on the inspected page.

## User preference overrides 
There's a new tool in Web Inspector to emulate user preferences. 
- Click the new icon in the Elements tab to reveal the User Preference Overrides popover.
- Set of toggles to override user preferences just for the inspected page while Web Inspector is open.
- These preferences map to CSS media features which you can use to adapt the style and behavior of your web page. 
- Some examples are these overrides are of the preference for color scheme maps to the ***prefers-color-scheme media feature in CSS*** and to ***override the preference for reduced motion.***

## Element badges 
- In the node tree view of the Elements tab, you can already see badges next to elements that act as CSS Flex or CSS Grid containers. Element badges provide a quick way to identify at a glance nodes of particular interest. In this case, nodes that create a CSS Grid or Flex layout context. `<element> grid` and `<element> flex`
- One of the trickiest CSS layout issues to debug is unwanted scroll, like a container that scrolls horizontally because the content inside it doesn't fit the available width. The new element badge `<element> Scroll` that helps to identify unwanted scroll.
- It provides a quick visual hint in the node tree when an element's content overflows its bounds and a scroll bar is applied.
- A new Event badge appears next to elements which have JavaScript event listeners attached to them. It works both for built-in events, like pointer or UI events, as well as custom JavaScript events that you dispatch in your code.


## Breakpoint enhancements
- When debugging JavaScript, you may be used to adding `console.log()` statements to your code. Breakpoints, on the other hand, are a powerful way to debug by pausing and stepping through JavaScript without having to make changes to your source.
- The easiest way to start is by clicking on a line number on a script file in the gutter of the Sources tab. 
- This sets a JavaScript breakpoint on that line of the script. Next time that line is about to run, Web Inspector will instead pause JavaScript execution at that point. While paused, you can observe the call stack, inspect the state of objects and variables in scope, and even make changes to them through the Console.
- You can resume JavaScript execution, or you can step through the code one expression at a time, using the stepping controls at the top. You can configure a breakpoint by right clicking on it and selecting Edit Breakpoint.
- There are many options you can set to control when the breakpoint is hit and even run actions when it does.
- You can control when a breakpoint is hit by setting a condition for it. This evaluates as JavaScript in the same scope where your breakpoint is set.
- You can also run actions when a breakpoint is hit, like evaluating a piece of JavaScript. 
- This runs in the same scope where the breakpoint is set. 
- You can use this to modify the state of your script before continuing. 
- You can also log messages to the console with expressions that have access to the state of variables and objects at the moment when JavaScript was paused. This is similar to adding a `console.log()` statement to your code, but without having to modify your source. Instead of logging variables and objects to the console, you can also use the Probe Expression action. This allows you to inspect the state of the given expression in the details sidebar panel of the Sources tab.

### Symbolic breakpoints
Symbolic breakpoints have been added  to pause before a function is about to be invoked. They are helpful to debug calls to built-in JavaScript functions or to pause in multiple functions with the same name.

If you want to learn more about these and the many other features you can use, go to [webkit.org](https://webkit.org) to find in-depth blog posts and documentation.

# Related Sessions
- [Adding a web development tool to Safari Web Inspector](https://developer.apple.com/documentation/safariservices/safari_web_extensions/adding_a_web_development_tool_to_safari_web_inspector)
- [Safari Technology preview](https://developer.apple.com/safari/download/)
- [Safari Release Notes](https://developer.apple.com/documentation/safari-release-notes)
- [Web Inspector Reference](https://webkit.org/web-inspector/)


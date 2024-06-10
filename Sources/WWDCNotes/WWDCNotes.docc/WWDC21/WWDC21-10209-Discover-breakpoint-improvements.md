# Discover breakpoint improvements

Breakpoints can help you debug issues by allowing you to pause and inspect problems in the middle of a process. Discover the latest improvements to breakpoints in Xcode including column and unresolved breakpoints. We’ll also go over best practices for general breakpoints and LLDB tips and tricks.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10209", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Source file breakpoints

When a compiler compiles a file under debug condition, it creates a map called a <kbd>line table</kbd> that maps source lines and columns to compiled addresses.
Source breakpoints rely on this table to determine when to pause the process execution.

- <kbd>Line breakpoint</kbd> is set in a single file
  - to create one is simply to click in the gutter next to the line you want to pause
- New: <kbd>Column breakpoints</kbd>
  - to create one, command-click on the expression to bring up the Actions popover, and then select <kbd>Set Column Breakpoint</kbd>

- When the breakpoint hits, Xcode uses the <kbd>line PC</kbd> to tell you the paused line, it draws a light green highlight over the line
- From Xcode 11.4, we also have <kbd>column PC</kbd>. The column PC shows you the paused column by drawing a green underscore under an expression.
  - it lets you know the expression the debugger is going to execute next

## Symbolic breakpoints

- breakpoints on function names that will pause the process when those functions are executed
- useful particularly when you don’t have access to the source files and hence, you can't compile them with debug info
- to create one, click on the <kbd>+</kbd> add button at the bottom of the breakpoint navigator, and select <kbd>Symbolic Breakpoint</kbd>
- LLDB will match the <kbd>symbol</kbd> name in your symbolic breakpoint in all the libraries that are loaded in the process, including the system libraries
  - to restrict the search scope, specify in which module you're interested in by using the <kbd>Module</kbd> field in your break point. A module is a binary or image that can be loaded during execution, including the main binary. (e.g. use `YourAppName` for the main module) 
  - if you're not sure on the symbol name, you can search for it:
    - in the source code
    - in lldb via `image lookup -rn insertSymbolNameHere` (`-r` for regex, `n` for name), you can also specify the module name after the symbol name

## Runtime issue breakpoints

- A runtime issue is an issue that occurs at runtime that is not as serious as a crash
  - by default, Xcode doesn't pause your process, because it can be too disruptive when you are focusing on something else
  - to pause execution when a runtime issue happens, create a Runtime issue breakpoint

- to create a runtime issue breakpoint, click on the <kbd>+</kbd> add button at the bottom of the breakpoint navigator, and select <kbd>Runtime issue Breakpoint</kbd>
- there are different types of runtime issue: for some of them to trigger, you need to enable the corresponding feature in the diagnostics tab of the scheme editor.
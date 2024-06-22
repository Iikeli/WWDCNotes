# Advanced Debugging with Xcode and LLDB

Discover advanced techniques, and tips and tricks for enhancing your Xcode debugging workflows. Learn how to take advantage of LLDB and custom breakpoints for more powerful debugging. Get the most out of Xcode’s view debugging tools to solve UI issues in your app more efficiently.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/412", purpose: link, label: "Watch Video (53 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample code](https://developer.apple.com/sample-code/wwdc/2018/UseScriptsToAddCustomCommandsToLLDB.zip)

## Advanced Debugging Tips and Tricks

- Configure behaviors to dedicate a tab for debugging
- Use auto-continuing breakpoints with debugger commands to inject code live
  - This help us debug changes without the need to edit your code and build and run our app over and over

- Create dependent breakpoints using `breakpoint set --one-shot true`
  - this creates a breakpoint that is automatically deleted after it's hit once

- Skip lines of code by: 
  - dragging the Instruction Pointer
  - send the command `thread jump --by 1`
    - this tells the debugger to jump over one line of code for the current thread

- object descriptions:
  - you can define your own objects descriptions by conforming to [`CustomDebugStringConvertible`][CustomDebugStringConvertible]
  - this `debugDescription` will be returned in the debugger when writing `po ...`
  - alternatively, you can use the debugger command `p ..`, which uses LLDB's built-in formatters to represent the object

- Pause when variables are modified by using watchpoints
  - a watchpoint is like a breakpoint, but it pauses the debugger the next time the value of that variable is changed

- Evaluate Obj-C code in Swift frames with`expression -l objc -O -- <expr>`
- Explore the view hierarchy of a view in the debugger via the private method `recursiveDescription`: 
  - ``expression -l objc -O -- [`self.view` recursiveDescription]``
  - The back ticks is like a step that says "first, evaluate the contents of this in the current frame and insert the result, and then we can evaluate the rest"

- create debugger command aliases for shortcuts via `command alias`
  - for example: `command alias poc "expression -l objc -O --"` - we can now use `poc <expr>` to run our command in an Objective-C context

- when changing UI/layout on the debugger, we don't see changes right away on the simulator because we're paused in the debugger. This is because Core Animation isn't currently applying any view module changes to the screen's frame buffer. To force these change to appear, we can use [`CATransaction.flush()`][flush] to tell Core Animation to update the screen's frame buffer.

- save your custom aliases into `~/.lldbinit`

LLDB Print Commands

| Command | Alias For | Does |
| --- | --- | --- |
| `po <expression>` | `expression --object-description -- <expression>` | evaluate `<expression>` and print its debug description |
| `p <expression>` | `expression -- <expression>` | evaluate `<expression>` and outputs its LLDB-formatted description |
| `frame variable <name>` | | read `<name>` value from memory and outputs its LLDB-formatted description |

[flush]: https://developer.apple.com/documentation/quartzcore/catransaction/1448270-flush
[CustomDebugStringConvertible]: https://developer.apple.com/documentation/swift/customdebugstringconvertible
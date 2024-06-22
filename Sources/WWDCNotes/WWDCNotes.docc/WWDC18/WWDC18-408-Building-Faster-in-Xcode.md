# Building Faster in Xcode

Build your apps faster in Xcode 10. Learn how to structure your projects and tweak your code to take full advantage of all processor cores. Whether you've made a few small code changes you want to give a try, or you're building your full app for release, these techniques will cut the time it takes to build a running app.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/408", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(rogerluan)
   }
}



## Increasing Build Efficiency via Architectural Changes

Sample dependency graph:

![][1Image]

- Parallelizing your build process:
    - The smaller your modules, the less likely it is for your dependencies to be built serially. In the example above, perhaps Physics don't need most of the utilities declared in Utilities, maybe just one or two functions. Extract those into a separate module, so that this new module and Utilities can be built in parallel, and Physics and start building as soon as this new module (now much smaller) finishes building, instead of waiting until Utilities finishes building.
    - Inspect your project for dead code, unlinking (or removing) unused dependencies.
    - Testing targets should have as few dependencies as possible: test only what's meant to be tested, and not code from other modules. Create separate testing targets for each of your modules, to improve parallelism when building testing targets.

- Measuring build time:
    - <kbd>Product</kbd> → <kbd>Perform Action</kbd> → <kbd>Build With Timing Summary</kbd>
    - Look for `PhaseScriptExecution` in the resulting logs after the build. If that phase is there on every build, it means your Run Scripts execution isn't being cached.
    - This setting is also available via `xcodebuild` CLI.

- Declaring Run Script input and output files:
    - Input files represent all the parameters your Run Script need to use to produce an output file;
    - Declare input and output files for your Run Scripts, so the compiler can cache results based on input/output and knows when it should re-run them;
    - Run Scripts are always run if they don't have input files;
    - Xcode 10 introduced a new `.xcfilelist` file to help us declare input and output files.

- Understanding dependencies in Swift:
    - When changing any interface in a module (e.g. creating, updating or deleting types, functions, protocols, properties, etc), all targets that depend on that module (e.g. the app's main target) will have to build all files again. (Note from WWDC note contributor: it's not clear from the session whether the compiler will cleverly only rebuild files that `import` the referred module. From what I interpreted, it's literally all the files.)
    - When changing only implementation of functions (not modifying interfaces), only the file being changed needs to be recompiled.

- Configurations:
    - Since Xcode 10 you can turn off `Whole Module` compilation mode for the `SWIFT_COMPILATION_MODE` build setting for Debug builds, due to compiler improvements. Delete this setting, so it goes back to the default, which is `Incremental`.

## Increasing Build Efficiency via Source Code Improvements

- Dealing with complex expressions:
    - Avoid using properties with `AnyObject` type, because the compiler will lookup the entire code base looking for matching function/property interfaces;
    - Break down complex expressions into smaller ones (this improves compiler performance and readability for developers);
    - Declare property types explicitly.

- Limiting your Objective-C ↔ Swift interface:
    - Keep your generated header minimal by using `private` on `IBAction`s, `IBOutlet`s, and `@objc` declarations;
    - If you're using `@objc` methods just to use them via `#selector`, opt for block-based APIs where possible.

[1Image]: WWDC18-408-dependency-graph

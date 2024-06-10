# Debug Swift debugging with LLDB

Learn how you can set up complex Swift projects for debugging. We'll take you on a deep dive into the internals of LLDB and debug info. We'll also share best practices for complex scenarios such as debugging code built on build servers or code from custom build systems.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110370", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What does LLDB need in order to show source code? 

- When the compiler compiles a function in a <kbd>.swift</kbd> file, it generates machine code (store in <kbd>.o</kbd> object files)
- on debug builds, <kbd>.o</kbd> files come with a `__debug_info` segment, which contains addresses in the executable that can be mapped to a source file and line number, and vice versa
- for archiving and distribution, debug info can be linked into <kbd>.dSYM</kbd> bundles
- the debug info linker is called `dsymutil`

## `image` and path remap 

- use `image list nameOfFramework` to check whether LLDB has found the debug dSYM of a third party framework embedded in our app
- use `image lookup 0xMemoryAddressHere` to get more info about the current address
- to remap source <kbd>.dSYM</kbd> paths, use `settings set target.source-map old/path new/path`
  - tip: instead of doing this at each session, create a lldb init file that is run at the beginning of each session (point to this file in your scheme)

- alternatively, each <kbd>.dSYM</kbd> bundle comes with a <kbd>UUID.plist</kbd> where we can set a `DBGSourcePathRemapping` dictionary

## Source path canonicalization

- We can instruct the compiler to canonicalize source paths before putting them into the debug info
- This is done using the `-debug-prefix-map` option
- This way the machine-specific path prefix can be replaced by a unique, canonical placeholder name that can then be remapped to the local path in LLDB

```shell
// Clang:
-fdebug-prefix-map $PWD=/BUILDROOT

// Swift:
-debug-prefix-map $PWD=/BUILDROOT
```

### po,p,expr vs v, frame

- LLDB is both a debugger and a compiler
- based on whether our LLDB command sides in the debugger or compiler side of LLDB, they might or or not fail
- (debugger side) `v` and `frame` get their type information from LLDB Debug Info (which in turn gets types from Swift reflection metadata_
- (compiler side) `p`, `po`, `expr` get type information from Modules
  - Modules are how the compiler organizes type declarations

How do we start diagnosing an issue that is happening on the compiler side? 

- new in Xcode 14, we have `swift-healthcheck` LLDB command
- it helps understanding if and why a module import failed
- it saves a <kbd>.log</kbd> of the Swift expression evaluator configuration

## How LLDB's compiler finds Swift modules?

- It's the build system's job to package up the modules so LLDB can find them
- Modules from system frameworks stay in the SDK (anyone can find them via `$ xcrun --show-sdk-path`), LLDB will find a matching SDK to read them from as it's attaching to your program
- when debugging straight from the <kbd>.o</kbd> object files, LLDB will find all non-SDK modules where they were at build time
- `dsymutil` can package a debug info archive (<kbd>.dSYM</kbd> bundle) for every dynamic library, framework or dylib, and executable
  - each .dSYM bundle can contain binary Swift modules, which may contain bridging headers <kbd>.h</kbd>, textual Swift interface files <kbd>.swiftinterface</kbd>, and most importantly, debug info.
  - for static archives, a Swift module needs to be registered with the linker (`ld ... -add-ast-path /path/to/My.swiftmodule`)
    - for dynamic libraries and executables, the build system will do this automatically for you. But for static archives, this is needed because static archives are not produced by the linker
    - Xcode's build system should do this for you, but you need to be aware if you have your own build system

You can check your build log to verify that that everything is linked, or use `dsymutil` to dump the symbol table of your executable and grep for "swiftmodule":

```shell
dsymutil -s MyApp | grep .swiftmodule
```

In Linux the swift driver supports a `-modulewrap` flag that converts binary Swift module files into objects that you can link into your binary together with the rest of the debug info. LLDB will find it there

```shell
swiftc -modulewrap My.swiftmodule -o My.swiftmodule.o
```

## Avoiding serialized search paths in Swift modules

- The Swift compiler will serialize Clang header search paths and other related options into the binary <kbd>.swiftmodule</kbd> files
- This is great, because it makes importing their Clang module dependencies just work during the build
- But when building on a different machine, these local paths can be detrimental
- Before shipping a binary <kbd>.swiftmodule</kbd> to another machine, set the `-no-serialize-debugging-options` compiler flag
- In Xcode this is controlled via the `SWIFT_SERIALIZE_DEBUGGING_OPTIONS` setting
- you can then reintroduce these search paths in LLDB with one of the following settings:

```shell
settings set target.swift-extra-clang-flags …
settings set target.swift-framework-search-paths …
settings set target.swift-module-search-paths …
```
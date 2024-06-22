# Behind the Scenes of the Xcode Build Process

Ever wonder what happens when you build your project in Xcode? Learn how Xcode automates the steps required to build an application, and go behind the scenes to learn how clang, swiftc, and the linker work together to turn your source code into a working program.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/415", purpose: link, label: "Watch Video (57 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Xcode 10 build system

- written from scratch in Swift
- improved performance and reliability

## Build Process: Execution of a collection of Tasks 

When building an app, we go throughout a sequence of steps:

- Compile and link source code
- Copy and process resources (e.g., headers, asset catalogues, storyboards)
- Code sign 
- (optional) custom work in shell scripts
- ...and more

Most of the tasks in the build process are performed by running command line tools: 

- `swiftc` (Swift compiler)
- `clang` (C, C++, ObjC compiler)
- `ld` (linker)
- `actool` (assets catalog tool)
- ...

These tools have to be executed with a very specific set of arguments, and in a particular order, based on the configuration of your Xcode project.

The build process automates the orchestration and execution of these tasks each time you perform a build.

The order in which build tasks are executed is determined from the dependency information of each task: 

- the inputs that a task consumes
- the outputs that a task produces

For example:

- a compilation task consumes a source code file (<kbd>.swift</kbd>, <kbd>.m</kbd>) as input, and produces an object file (<kbd>.o</kbd>) as output
- a linker task consumes a number of object files (<kbd>.o</kbd>) produced by the compiler in previous tasks, and produces and executable or library output

If tasks are independent, they can (and will) run in parallel.

## What happens when you press build?

1. the build system takes the build description in your Xcode project file, parses it, takes into account all the files in your project, your targets, the dependency relationships, your build settings, etc.
2. the build system will generate a Directed Graph (from the info of step 1), which represents all the dependencies between the input and output files in your project. It will generate the tasks that will be executed to process them.
3. the low-level execution engine (`llbuild`, [it's open source][llbuild]) processes this graph, looks at the dependency specifications and figures out which tasks to execute, the sequence or order in which they must be run, which tasks can be run in parallel. Then proceeds to execute them. 

### Discovered dependencies

When `llbuild` runs, besides outputting the object file (<kbd>.o</kbd>), it might also output a listing file (<kbd>.d</kbd>), containing which header files (imports) were included by that source file.

The next time you build, the build system uses the information from this file (<kbd>.d</kbd>) to make sure that it recompiles the source file if you change any of the header files that it includes.

Having an accurate dependency information (via <kbd>.d</kbd>) is very important in order for incremental builds to work correctly and efficiently.

### Change Detection and Task Signatures

- Each task in the build graph has an associated signature
- this signature is a hash computed from various task information
  - the stat info of the task's inputs
  - file paths
  - modification time stamps
  - the command line indication used to perform the command
  - and other task-specific metadata such as the version of the compiler that's being used

- build system tracks task signatures among multiple builds
- a task is re-run when the hash is different between builds, it's skipped otherwise

### Where do dependencies come from?

- Built in - the various tools within the build system know which tasks accept which input and produce which output (e.g. `atool`, `ld`, ..)

- Target dependencies
  - before Xcode 10, when a target was built, it required the compilation of the entire dependent target to be completed before it could start
  - with Xcode 10 new build system, targets can start building sooner
  - all run script phases need to complete before this parallelization can take effect

![][targetDep]

- Implicit dependencies
  - if you list a target in your <kbd>Link library With Binaries</kbd> build phase, and <kbd>Find Implicit Dependencies</kbd> are enabled in the scheme editor (on by default), the build system will establish an implicit dependency on that target (even if it's not listed in target dependencies)

![][linkDep]

- Build phase dependencies
  - each phase is a separate task
  - the build system might run these phases in a different order (e.g. if <kbd>Link Binary with libraries</kbd> phase is ordered before <kbd>Compile sources</kbd>)
  - a wrong phase order might cause a build failure

![][buildDep]

- Scheme order dependencies
  - If you have the <kbd>Parallelize Build</kbd> check box enabled in your scheme settings, you get better build performance and the order of your targets in your scheme doesn't matter
  - if you turn <kbd>Parallelize Build</kbd> off, Xcode will attempt to build your targets in the order you listed them in the build action of the scheme one by one

![][schemeDep]

### Tips

- set input/outputs on your build phases (lets the build system avoid re-running the script tasks unnecessarily)
- Avoid Auto-link for project dependencies
  - this setting allows the compiler to automatically link to the frameworks corresponding to any modules you import without having to explicitly link them in your link library's build phase
  - auto-link does not establish dependency on that framework at the build system level (won't guarantee that the target you depend on is actually built before you try to link against it)
  - you should rely on this feature only for frameworks from the platform SDK (UIKit, etc.).

![][autolink]

- 
  - For targets in your own projects, make sure to add explicit library dependencies

![][noAutoLink]

## Clang Compiler

Clang is Apple's official compiler for the C language family.

### Header Maps

- tell the build system (Clang) where the file headers are located 
- their purpose is to point back to your source code
- if needed, they prepend your framework name to headers, e.g. <kbd>Cat.h</kbd> will change to <kbd>PetKit/Cat.h</kbd>, however this can cause issues with Clang modules
  - it is recommended that you always specify the framework name when you include a public or private header file from your own framework (instead of writing `import <Cat.h>` or `import Cat.h`, write `import <PetKit/Cat.h>` or `import PetKit/Cat.h`)
  - you don't need to do this for project headers

How does the build system find the system header maps?

- header maps are only for your code
- Clang searches system header files in two folders be default:

```cli
$(SDKROOT)/usr/include 
$(SDKROOT)/System/Library/Frameworks (framework directory)
```

> `$(SDKROOT)` specifies the directory of the base SDK to use to build the product.

Let's say we're searching for `Foundation`'s header map:

1. first Clang searches `$(SDKROOT)/usr/include/Foundation/Foundation.h`, it won't find it there (unless you've declared a new Foundation header, this is called header shadowing)
2. then it will try to find it in the `$(SDKROOT)/System/Library/Frameworks` path, in this folder Clang needs to search for the framework and then dive into it and grab the headers:

```cli
$SDKROOT/System/Library/Frameworks/Foundation.framework/Headers/Foundation.h
```

If it cannot find it there, it will try to search for the header in the `PrivateHeaders` directory:

```cli
$SDKROOT/System/Library/Frameworks/Foundation.framework/PrivateHeaders/Foundation.h
```

Apple doesn't ship any `PrivateHeaders` but other projects might, hence Clang will look there, too. 

### Clang modules

Clang modules are on-disk cached header representations that allow us to only find and parse the headers once per framework.

To make this possible, every module needs to have certain properties:

- context-free: ignores the context-related statement (e.g., macro definitions)
- self-contained: has to specify its own dependencies 

How does Clang know when it should build a module?

#### System modules

- first Clang has to find the header in the framework (as we saw above).
- then Clang will look for a `Modules` directory and a `module.modulemap` relative to the header's directory. A Module Map describes how a certain set of header files translate onto your module

Here's the whole Foundation module map:

```objc
// Foundation.framework/Modules/module.modulemap 

framework module Foundation [extern_c] [system] { 
  umbrella header "Foundation.h" 

  export *
  module * { 
    export * 
  }

  explicit module NSDebug { 
    header "NSDebug.h" 
    export * 
  }
}
```

The module map contains the framework name and specifies what headers are part of the module.

In Foundation's case, `Foundation.h` is an umbrella header, which also needs to be parsed to look for other headers in the module.

Lastly, we use the command `$ clang -fmodules ...` to create a module cache on disk. Note that creates a cache folder by hashing the command arguments, if we change the `clang -fmodules ...` invocation, a new cache will be used/generated.

#### 3rd party modules

- In our frameworks, the header map will point to the source directory, where we don't have a `Modules`
- Clang doesn't know how to handle this
- solution: Clang Virtual File System (VFS)
  - VFS creates a virtual abstraction of a framework that allows Clang to build the module
  - the abstraction just points to the files back in your directory

### Clang and Swift

As Swift files don't have headers, the compiler has to perform some additional bookkeeping.

The compiler has to: 

1. find declarations both within Swift targets and from Objective-C (to compile Swift files depending on Objective-C declarations)
2. generate interfaces describing the contents of each file (to compile Objective-C files depending on Swift declarations)

#### Finding declarations within a Swift Target 

- unlike Clang, when compiling one Swift file (with `swiftc`), the compiler will parse all the other Swift files in the target. To examine the parts of them that are relevant to the interfaces.
- note that it will only parse the files headers, not the implementations
- in Xcode 9, this resulted in repeated work as the compiler had to repeatedly parse all files (once to produce the associated <kbd>.o</kbd>, and multiple times as an interface to find declarations)
- in Xcode 10, the compiler gather multiple files into groups (that will be compiled in separated processes) to share as much work as possible (avoids repeating parsing within a group, only need to parse again across groups/processes).  

#### Finding declarations from Objective-C

- `swiftc` embeds `clang` and uses it as a library 
- this makes it possible to directly import Objective-C (without having to manually create a Swift interface for each declaration)
- in any target, when you import an Objective-C framework, the importer finds declarations in the headers exposing Clang's module map for that framework
- within a framework that mixes Swift and Objective-C code, the importer finds declarations in the **umbrella header** (this header defines the public interface, this way, Swift code inside the framework can call public Objective-C code in the same framework. The umbrella header name is the name of the framework, e.g. `MyFramework.h`)
- within app and unit test bundles, via the target **bridging header** (which allows Objective-C declarations to be called from Swift) 

Clang Importer makes methods more Swifty, e.g.:

- methods that use the `NSError` idiom become throwing methods in Swift
- methods will drop parameter type names following verbs and prepositions (see full list [here](https://github.com/apple/swift/blob/82568468494fd74dba17ba695c3cdf6fab1c3369/lib/Basic/PartsOfSpeech.def))
- use `NS_SWIFT_NAME` to change imported name explicitly
- if you'd like to check/preview how your Objective-C code will be imported into Swift: go to Xcode's related items (<kbd>^</kbd> + <kbd>1</kbd>) and select <kbd>Generated interface</kbd>

#### Generate interfaces to use in Objective-C

- Swift generates a header that you can `#import`
- this allows you to write classes in Swift and call them from Objective-C

Declaration in Swift:

```swift
class PetCollar: NSObject {
  @objc var color: UIColor = ...
  var name: String = "..."
}
```

Use from Objective-C:

```objc
#import "PetWall-Swift.h"

- (void)resetCollarColor {
  self.collar.color = [UIColor redColor];
}
```

The compiler will generate Objective-C declarations for Swift classes extending `NSObject` and methods/properties marked `@objc`

Swift declaration:

```swift
class PetCollar: NSObject {
  @objc var color: UIColor = ...
  var name: String = "..."
}
```

Generated Objective-C header:

```objc
SWIFT_CLASS("_TtC7PetWall9PetCollar")
@interface PetCollar: NSObject
@property (nonatomic, strong) UIColor * _Nonnull color; 
- (nonnull instancetype)init; 
@end
```

- the compiler ties the Objective-C class to the mangled name of the Swift class, which includes the name of the module (PetWall): this prevents conflict when two modules define classes with the same name
  - Use `@objc(YOUROBJCCLASSNAMEHERE)` to provide a Objective-C custom name

#### Generate interfaces to use in other Swift Targets

What's a module?

- distributable unit of declarations
- must first import other modules to see their declarations
- can import Clang/Objective-C modules (e.g., XCTest)
- in Xcode, each Swift target produces a separate module (including your app target, this is why you have to import your app's main module in order to test it from your unit tests)
- when importing a module, the compiler deserializes a special Swift module file (<kbd>.swiftmodule</kbd>) to check the types when you use them

Expose Declarations to Other Modules via <kbd>.swiftmodule</kbd>:

- Serialized, binary representation of module’s declarations
- Includes bodies of `@inlinable` functions
- Includes private declarations (for debugging)

For incremental builds: 

- the compiler produces partial Swift module files and then merges them into a single file that represents the contents of the entire module
- this merging process also makes it possible to produce a single Objective-C header (out of the final <kbd>.swiftmodule</kbd>)

## Behind the Scenes of the Xcode Build Process: Linking

### The Linker

- it's the final task in building an executable Mach-O
- it combines the output of all compiler invocations (Object files <kbd>.o</kbd>, Libraries <kbd>.dylib</kbd>, <kbd>.tbd</kbd>, <kbd>.a</kbd>) into a single file
- moves and patches code generated by the compilers (it cannot create code)

### Symbols

- A symbol is a name to refer to a fragment of code or data
- Fragments of code may reference other symbols (e.g., when a function calls another function)
- Symbols can have attributes on them that alter the linker’s behavior
  - e.g. Weak is one of those attributes
    - weak symbols state that the symbol might not be there when we run the executable on the system
    - this is what all the availability markup that says this API is available on iOS x 

- Languages often encode data into a symbol mangling the symbol

### Object Files (.o)

- Output of individual compiler actions
  - they are collections of code and data we refer via symbols
- A non-executable Mach-O file containing code and data fragments
  - while they are compiled code, they have bits missing, which is what the linker is going to glue together

- Each fragment in these <kbd>.o</kbd> is represented by a symbol
- Fragments may reference “undefined” symbols
  - when a <kbd>.o</kbd> refers to a function in another <kbd>.o</kbd> file, the linker is responsible to find those undefined symbols and link them

### Libraries

Libraries define symbols that are not built as part of your target

- Dylibs: Dynamic libraries (<kbd>.dylib</kbd>)
  - Mach-O file that exposes code and data fragments executables can use
  - some of those are distributed as part of the OS, that's what Apple's frameworks are
  - can also be from 3rd parties

- TBDs: Text Based Dylib Stubs (<kbd>.tbd</kbd>)
  - only contains symbols
  - no bodies for any symbols, just their names (no binary code)
  - used for distributing Apple's SDKs to reduce their size (it reduces the size of the SDK you download with Xcode, not the size of the app you're building)

- Static archives (<kbd>.a</kbd>)
  - archive of multiple <kbd>.o</kbd> files built with the `ar` tool
  - it's just a <kbd>.zip</kbd> file with <kbd>.a</kbd> extension (<kbd>.a</kbd> was the original archive format used by UNIX)
  - only <kbd>.o</kbd> files with symbols you reference are included in your app
    - If you're using some sort of non-symbol behavior like a static initializer, or you're re-exporting them as part of your own dylib, you may need to explicitly use something like force load or all load to the linker to tell it bring in everything 

[schemeDep]: WWDC18-415-schemeDep
[targetDep]: WWDC18-415-targetDep
[linkDep]: WWDC18-415-linkDep
[autolink]: WWDC18-415-autolink
[noautolink]: WWDC18-415-noautolink
[buildDep]: WWDC18-415-buildDep

[llbuild]: https://github.com/apple/swift-llbuild

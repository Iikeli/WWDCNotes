# Meet mergeable libraries

Discover how mergeable libraries combine the best parts of static and dynamic libraries to help improve your app’s productivity and runtime performance. Learn how you can enable faster development while shipping the smallest app. We’ll show you how to adopt mergeable libraries in Xcode 15 and share best practices for working with your code.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10268", purpose: link, label: "Watch Video (26 min)")

   @Contributors {
      @GitHubUser(marcolg97)
   }
}



> Mergeable Libraries is a new model for building and distributing libraries powered by the static linker

## Static vs Dynamic libraries

### Static libraries: 
* Static libraries are a collection of object files. 

* At build time, the static linker finds which APIs to use from those libraries 
and copies that code into the app binary -> since it's copied, *the library isn't needed after building*.

* Build time slowdown if code in static libraries changes or if more libraries are used -> makes iterative building and debugging slower. 

### Dynamic libraries (dylibs)
* They are the binary file type for framework targets in Xcode.

* Faster builds -> the static linker does not need to copy code

* The static linker records the installed path of the library into the app binary for later

* Any frameworks not in the Apple SDK must be embedded into the app bundle. This leads an increase in memory consumption and app launch time because generally, we have hundreds of frameworks. 

![Static vs Dynamic libraries][static-vs-dynamic]

[static-vs-dynamic]: WWDC23-10268-static-vs-dynamic


When an app is launched, the dynamic linker named dyld must find and load framework dependencies including libraries those frameworks depend on. As more are used, this results in a steady increase in memory consumption and app launch time

And when you factor in dependencies from the Apple SDK, apps can often load hundreds of frameworks. Our platforms have heavily optimized system libraries to account for this but this doesn't apply to frameworks that get embedded in apps. 

![Static vs Dynamic libraries][dynamic-linking]

[dynamic-linking]: WWDC23-10268-dynamic-linking

 
### Recap

> There are some tradeoffs when deciding between using static and dynamic libraries.
> While dynamic libraries have little impact on build time but noticeable launch time consequences, static libraries provide minimal launch time impact but are costly on build time

Due to this, we have historically recommended measuring what's best for your app. With mergeable libraries, this is no longer needed because
**mergeable libraries unlock the best of both linking strategies**

![Static vs Dynamic libraries][recap]

[recap]: WWDC23-10268-sweet-spot

## Meet mergeable libraries: how mergeable libraries work to make your apps build and run faster.

* Any dynamic library can be built as mergeable.

* Consider any binary image (like an executable) and the frameworks this binary depends on are given to the static linker. These dependencies can become **mergeable libraries** and the linked output can become the **merged binary**. 

Static linker generates **metadata** when creating the library (use option -make_mergeable). The metadata is within the binary, increasing its overall size. It allows the linker to treat the library similarly to a static library when it's used as a link dependency.

* With the metadata, users of the library can choose to statically link as normal dynamic libraries or merge them. 

* The merged binary output can be an executable, like an app, or another dynamic library, like a framework. 

* Merging is comparable to how static libraries get linked. In the end, you're left with a **binary that contains the segments of the libraries**. And that output binary remains the same file type (-merge_library or -merge_framework). 

![Static vs Dynamic libraries][mergeable-libraries]

[mergeable-libraries]: WWDC23-10268-mergeable-libraries

### But how is merging better than just linking? 

* Smaller overall app bundle
    * Metadata isn't needed and can be removed after they've been merged
    * The linker can de-duplicate content (it removes redundant symbol references**, Objective-C selectors, and objc_msgsend stubs)

* The image type of the final binary remains the same too. That means any already supported linker optimizations can be applied.

* Positive impact on app launch because fewer frameworks are loaded
    * Reduces the work dyld and the kernel need to do when launching your app 
    * Reduces memory usage


Mergeable libraries make this possible with minimal code and configuration changes. And this scales nicely as you adopt newer frameworks. 

Let's revisit the earlier diagram about dynamic linking. All of these embedded frameworks can become mergeable since the linker can generate metadata for them. 

We can create a framework that merges the contents of the other libraries. So you end up with **only one framework to embed**. 

Dyld only needs to load that one library containing all segments across the embedded frameworks. Merging, in this way, can greatly simplify large dependency chains. 

![Static vs Dynamic libraries][mergeable-libraries-2]

[mergeable-libraries-2]: WWDC23-10268-mergeable-libraries-2

## Using mergeable libraries: how to enable mergeable libraries in Xcode 15

There are two ways to enable library merging in Xcode. 

### Automatic merging
* Merge all direct dependencies that are embedded framework targets
* Works well on app targets
* Libraries link directly into app binary -> similar launch time performance to static libraries

* With Automatic merging the three forest frameworks will become mergeable (except SwiftUI since it's a system library)
* These frameworks will be merged directly into the app binary. These frameworks won't be needed at launch and can be removed from disk

To enable Automatic merging inside Xcode 15 go to Build Settings, search for MERGED_BINARY_TYPE and change "Create Merged Binary" to "Automatic". The section with these settings is called "Linking - Mergeable Libraries". 

**The exports of mergeable libraries are preserved in the app**. It's often not applicable that apps export symbols and it negatively impacts the size and build time. To prevent this, use the linker option -no_exported_symbols. This can be applied in Xcode by updating "Other Linker Flags" with "-Wl, -no_exported_symbols." 

![Static vs Dynamic libraries][automatic]

[automatic]: WWDC23-10268-automatic

###  Manual Merging
* Fine-grained approach to specifying the libraries to merge (when only some of your frameworks should be merged together). 
* When some dependencies need to stay in the app bundle. 

It is enabled by setting MERGED_BINARY_TYPE = manual on the overarching target. The libraries that should make up the final merged product are recognized by setting MERGEABLE_LIBRARY to YES and for libraries that should stay on disk, keep the default setting of MERGEABLE_LIBRARY to NO. 

Consider now we have a XCTest target that depends on the Forest framework too; another example is when we have targets like app extensions that create a similar-looking dependency graph.

* First I'll isolate the app dependencies for the three Forest frameworks.
* Then I'll create a framework, ForestKit, that merges the libraries but will also satisfy my test dependency. ForestKit is considered a group library because it'll encapsulate the mergeable libraries both my app and tests depend on. 
    * Click on "+" on XCode 15 above Targets section and select Framework template under macOS tab.
    * Under Link Binary With Libraries, add all the three frameworks
    * Go to Build Setting, "Create Merged Binary" and set the value to Manual 
* As I'm enabling manual mode, I'll need to explicitly set which frameworks to make mergeable (ForestBuilder, ForestUI, and Forest). 
    * For each framework I want to merge, go to the framework target, select Build Setting and set Build Mergeable Library to YES.
* These dependencies will be merged inside ForestKit


![Static vs Dynamic libraries][manual]

[manual]: WWDC23-10268-manual

I'm finished creating my merged ForestKit framework. But I need to update some dependencies. Because I've created a framework that encapsulates most of my dynamic libraries, I need to ensure my app and **tests link against ForestKit and not the others**. 
 * Go inside the project target, select Build Phases and under "Link Binary with Libraries" remove the three frameworks and leave only the ForestKit. Do the same with the text target and add ForestKit if not present. 


## Debug Mode
* Only during the release mode the libraries are merged then removed from disk
* This prevents the build time overhead due to the merge when we are developing(like the static library)

> **To support iterative development in Xcode, the linker will not merge in debug mode**. The build system tells the linker to re-export the libraries instead. Reexporting is a linker option that allows the implementation of code to live in one dynamic library but has it show up as if it's implemented in another. 

* In other words, this means all of the libraries' APIs are reachable by just depending on the merged target, like your app extensions or tests. 
* This results in a similar build time benefit as with dynamic libraries. 

### Symbolication
Symbolication is the process of associating these machine instructions back to the original source code. This is useful to be able to understand crash logs or to profile and debug your code. 

How does this work with merged binaries? 

> When you enable merging, source location information is still preserved from the original library. **That means your debugging experience remains the same**. But keep in mind, when library information is displayed, like for stack traces, it will **show the path to the merged binary**. This information is presented in crash logs, inside Instruments, and in the debugger. 

### Considerations for mergeable libraries: considerations and what we recommend when using mergeable libraries

1. Mergeable libraries as dependencies

    If some dependencies are not merged and they point to the merged framework, they need to update to depend on the merged framework, because mergeable ones are removed from disk. 

2. Autolinking with mergeable libraries

    Autolinking finds frameworks from module imports; so if you're importing a module from a mergeable library, **this could cause dynamic linking issues**. The solution is not to disable autolinking but link against the merged framework.
    * Go to the project target, select Build Phase and add in the "Link Binary with Libraries" and remove the mergeable ones if it's there already.

3. APIs for runtime lookup

    If you use dynamic linking APIs like dlopen, those input paths will also need to point to the merged framework target.

4. The new static linker

    Similarly, resource lookup could be impacted by library merging. This is because bundle is an API to have the runtime load a framework's bundle.

    Prior to iOS 12, framework binary was needed for lookup. In iOS 12, a hook was added to enable lookup for this scenario, so you rely on bundle lookup support. So your project must have iOS 12+ target in order to use mergeable libraries. 

    You can disable with a linker option -no_merged_libraries_hook. Add this option anyway to improve launch time performance.
    
    Inside the toolchain there are two static linkers (new and old linking). The older linker is still supported for backwards compatibility. New linker doesn not support armv7k so the minimum deployment target is watchOS 9+.

5. Distributing frameworks

    You can ship a mergeable library by creating an XCFramework in the Swift Package Manager or in Xcode. This allows you to build the framework including its metadata for distribution. When other developers use the framework, they can decide whether to enable merging. 

    Mergeable metadata roughly doubles the size of the dylib. This doesn't impact the size of an app because metadata is discarded along with the mergeable library after building the app. Otherwise, that metadata does get stripped to prevent bloat when embedding them in apps. 



### Recommendations

* Link against merged target throughout the project
* Update input in script phases to the merged binary
* Link any mergeable libraries into merged binary
* Metatada is stripped if not merged


### Wrap-up
* Mergeable libraries offer convenience and flexibility. 
* With automatic and manual workflows, you can restructure and add mergeable libraries at your leisure and leave the necessary ones on disk. 
* You can gradually adopting or profiling. 
* Ensure all dependents of those libraries are relying on the merged binary instead of the libraries that get removed. 

> Mergeable libraries offer size, build, and runtime improvements when applied to framework and executable targets. 


For documentation about mergeable libraries, review *"Configuring your project to use mergeable libraries."*

And to learn more about static and dynamic linking, check out the session *"Link fast: Improve build and launch times."* 



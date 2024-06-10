# Develop advanced web content

Develop in JavaScript, WebGL, or WebAssembly? Learn how the latest updates to Safari and WebKit — including language changes to class syntax — can help simplify your development process, enhance performance, and improve security. We’ll explore several web APIs that can help provide better interoperability and bring new capabilities to your web content.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10030", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## JavaScript enhancements

- Class field syntax
  - Private instance fields and methods
    - use `#variableName` instead of `_variableName`
    - define methods with `#methodName` instead of methodName
    - applies to static fields

  - Language-enforced access protection
  - Public and private static fields

- Weak references
  - Weakly refer to an object without preventing garbage collection: use `new WeakRef(object)` to held a weak reference to an object
  - Get underlying object from weak reference if it exists: check via `deref()?` e.g. `weakObject.deref()?.objectMethod();`
  - Register notifications about objects garbage collection via `FinalizationRegistry` (warning: you may not get the callback from FinalizationRegistry right way because it runs on event loop)

- Top-level await
  - available in modules
  - use `await` outside of a async function at the top level
  - async modules block execution of modules importing them 

- Module in workers
  - efficient resource utilization
  - ergonomic and performance benefits of modules (dynamic import, optimized loading and execution, and dependency management)
  - easy to move heavy work to background thread
  - modules are now available in different types of workers, including web worker, service worker, and worklet

- Internationalization improvements

## WebAssembly updates

What's WebAssembly?

- binary instruction format for a stack-based virtual machine
- type of a code that can be run in modern web browsers with performance close to native code
- designed to be a portable compilation target for programming languages like C, C++, or Rust

Enhancements:

- bulk memory operations
- non-trapping float-int conversions
- Sign-extension operators
- JavaScript BigInt integration
- reference types
- streaming download and compilation

## New web APIs

New features/feature updates:

- WebGL 2.0
- Storage Access
- WebM
- VP9
- Media Recorder
- Audio Worklet
- Web Share
- Media Session
- Speech recognition

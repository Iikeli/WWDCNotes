# Improve app size and runtime performance

Learn how we've optimized the Swift and Objective-C runtimes to help you make your app smaller, quicker, and launch faster. Discover how you can get access to efficient protocol checks, smaller message send calls, and optimized ARC simply when you build your app with Xcode 14 and update your deployment target. 

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110363", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Protocol checks

Example:

```swift
// 👇🏻 Protocol definition
protocol CustomLoggable {
  var customLogString: String { get }
}

func log(value: Any) {
  if let value = value as? CustomLoggable { // 👈🏻 protocol check
    ...
  }
}
```

- Whenever possible, this check is optimized away at build time, in the compiler
- However, we don't always have enough information yet, so this check often needs to happen in the runtime, with the help of protocol check metadata
- With this metadata, the runtime knows whether this particular object really does conform to the protocol
- Part of the metadata is built at compile time, but a lot can only be built at launch time, particularly when using Swift generics
- on apps that rely heavily in Swift, this could add up to half the launch time

New this year:

- protocol checks are pre-computed as part of the dyld closure for the app executable and any dylib it uses at launch (a.k.a. this is done at app installation time)
- enabled even for existing apps when running on iOS 16, tvOS 16, or watchOS 9.

## (ObjC) Message send

- with new compilers and linker in Xcode 14,message send calls are up to <kbd>8 bytes</kbd> smaller, down from <kbd>12 bytes</kbd>, on ARM64
- binaries up to 2% smaller overall
- enabled by Xcode 14, even when targeting an older OS release as deployment target
- Defaults to balanced performance and size optimization
- Opt into optimizing for size only using `-Wl,-objc_stubs_small` linker flag

### How it's done: Selector stubs

Prior:

- whenever we interact with objc models, we almost always end up needing an instruction to call `objc_msgSend`, even when doing property accesses
- this is because at compile time, we don't know which method to call, therefore we ask objc runtime using `objc_msgSend` to find the right method

```objc
NSCalendar *cal = [self makeCalendar];                              // 👈🏻 compiler emits bl _objc_msgSend
NSDateComponents* dateComponents = [[NSDateComponents alloc] init]; // 2x 👈🏻 compiler emits bl _objc_msgSend
dateComponents.year = 2022;                                         // 👈🏻 compiler emits bl _objc_msgSend
dateComponents.month = 6;                                           // 👈🏻 compiler emits bl _objc_msgSend
dateComponents.day = 6;                                             // 👈🏻 compiler emits bl _objc_msgSend
NSDate *theDate = [cal dateFromComponents:dateComponents];          // 👈🏻 compiler emits bl _objc_msgSend
return theDate;
```

- to tell the runtime which method to call, we have to pass a selector to these `objc_msgSend` calls. This needs two more instructions to prepare the selector, each of these instructions takes 4 bytes on ARM64.

Example: 

```objc
// ObjC
NSDate *theDate = [cal dateFromComponents:dateComponents];

// 👇🏻 compiler emits
adrp x1 [selector "dateFromComponents"]     // 👈🏻 4 bytes on ARM 64
ldr  x1 [x1, selector "dateFromComponents"] // 👈🏻 4 bytes on ARM 64
bl _objc_msgSend
```

- in conclusion, for each `objc_msgSend` call, we're using 12 bytes (4B for `_objc_msgSend`, 8B for the selector) 

What's new:

- for any given selector, it's always the same code, example from above:

```objc
adrp x1 [selector "dateFromComponents"]     // 👈🏻 always the same instruction for dateFromComponents
ldr  x1 [x1, selector "dateFromComponents"] // 👈🏻 always the same instruction for dateFromComponents
bl _objc_msgSend
```

- we can share this same code and only emit it once **per selector** instead of at every message send
- we do this via a helper function, and call that function instead
- this function is called Selector Stub

Therefore we go from this:

```objc
// ObjC
NSDate *theDate = [cal dateFromComponents:dateComponents];

// 👇🏻 compiler emits
adrp x1 [selector "dateFromComponents"] 
ldr  x1 [x1, selector "dateFromComponents"]
bl _objc_msgSend
```

to this:

```objc
// ObjC
NSDate *theDate = [cal dateFromComponents:dateComponents];

// 👇🏻 compiler emits
bl _objc_msgSend$dateFromComponents

// Where the _objc_msgSend$dateFromComponents Selector stub is defined ONCE per program
// Selector stub:
_objc_msgSend$dateFromComponents:
adrp x1, [selector "dateFromComponents"]
ldr  x1, [x1, selector "dateFromComponents"]
b    _objc_msgSend
```

However `_objc_msgSend` still needs to jump to the actual message send code (the Call stub), therefore we're doing two jumps in our machine code (one to go to the Selector stub, one to the Call stub)

Based on the optimization that we would like, we can merge the Selector and Call stub (making only one jump), or we can keep them separate. This is the difference between the default behavior (balanced performance and size optimization) and the optimized for size one (`-Wl,-objc_stubs_small` linker flag):

```objc
// Separate selector and symbol stubs
// Optimize for Size
// Enable using -Wl,-objc_stubs_small

bl _objc_msgSend$dateFromComponents // 👈🏻 jump 1

// Selector stub
_objc_msgSend$dateFromComponents:
adrp x1, [selector "dateFromComponents"]
ldr  x1, [x1, selector "dateFromComponents"]
b    _objc_msgSend                  // 👈🏻 jump 2

// Call stub
_objc_msgSend:
adrp ...
ldr  ...
br   ...
```

vs.

```objc
// Combined selector and symbol stubs
// Balanced Size/Performance
// Enabled by default

bl _objc_msgSend$dateFromComponents // 👈🏻 jump 1

// Where the _objc_msgSend$dateFromComponents Selector stub is defined ONCE per program
// Selector stub:
_objc_msgSend$dateFromComponents:
adrp x1, [selector "dateFromComponents"]
ldr  x1, [x1, selector "dateFromComponents"]
adrp ...
ldr  ...
br   ...
```

## Retain and release

- retain/release calls are now up to 4 bytes smaller, down from 8 on ARM64
- binaries up to 2% smaller overall
- enabled by new compilers in Xcode 14
- requires new runtime support, hence your deployment target must be iOS 16, tvOS 16, or watchOS 9

### How it's done

- thanks to ARC (automatic reference counting), our code compiles into a lot of retain/release calls
- whenever we make a copy of a pointer to an object, we need to increment its retain count to keep it live
- we do that by calling into the runtime, using `objc_retain`
- when our variables go out of scope, we then need to decrement the retain count using `objc_release`

```objc
NSCalendar *cal = [self makeCalendar];                              // 👈🏻 bl _objc_retain
NSDateComponents* dateComponents = [[NSDateComponents alloc] init]; // 👈🏻 bl _objc_retain
dateComponents.year = 2022;                                         
dateComponents.month = 6;                                           
dateComponents.day = 6;                                             
NSDate *theDate = [cal dateFromComponents:dateComponents];          // 👈🏻 bl _objc_retain
return theDate;
                                                                    // 👈🏻 bl _objc_release
                                                                    // 👈🏻 bl _objc_release
                                                                    // 👈🏻 bl _objc_release
```

> in actuality, the compiler will do some optimizations that will avoid some of these calls

- `objc_retain`/`objc_release` functions are just plain C functions
  - they take a single argument, the object to be released
  - with ARC, the compiler inserts calls to these C functions, passing the appropriate object pointers
  - Because of that, these calls have to respect the C calling convention, defined by our platform ABI (Application Binary Interface)
  - this results in extra `move` instructions just for passing the pointer in the right register

Optimization:

- By specializing retain/release with a custom calling convention, the system can opportunistically use the right variant depending on where the object pointer already is, meaning we don't need the extra `move` instructions
- skipping these `move` instructions saves the 4 bytes

## Quicker Autorelease elision

- enabled by ObjC runtime changes (automatically happens when running iOS 16, tvOS 16, watchOS 9, or macOS 13)
- with additional compiler changes, the app binary is also smaller (requires deployment target must be iOS 16, tvOS 16)

What is Autorelease elision?

- When we call a method that returns a value, we call `retain` on the returned object
- on the method's body, we don't call `release` on the returned object, as that would release the object from memory
- instead we call `autorelease`, so the method caller can retain it

```objc
                                                // 👇🏻 retain call
myValue = [[someInstance aMethodReturningAValue] retain];

-(MyType *)aMethodReturningAValue {
  ...

  return [newValue autorelease]; // 👈🏻 we don't release, we autorelease, so the caller (above)
                                 //    can retain it before the instance gets released.
}
```

- `autorelease` tells the runtime that we're returning an object that will immediately be retained
- thanks to some improvements in both compiler and Objc runtime, `autorelease` overhead is now cheaper, and with clever use of pointers is faster as well
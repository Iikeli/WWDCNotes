# Refine Objective-C frameworks for Swift

Fine-tune your Objective-C headers to work beautifully in Swift. We’ll show you how to take an unwieldy Objective-C framework and transform it into an API that feels right at home. Learn about the suite of annotations you can use to provide richer type information, more idiomatic names, and better errors to Swift. And discover Objective-C conventions you might not have known about that are key to a well-behaved Swift API.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10680", purpose: link, label: "Watch Video (42 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



`swiftc`'s Clang Importer automatically generates interfaces for Objective-C frameworks, for example:

- `NSString` and `NSDate` parameters are bridged into `String` and `Date` structs
- all `init` methods are imported as initializers
- all methods are rewritten into a style closer to Swift
  - methods that follow the Objective-C error handling convention are turned into throwing functions

For example, we go from:

```objc
// SKMission.h 

#import <Foundation/Foundation.h>
#import <SpaceKit/SKAstronaut.h>
#import <SpaceKit/SKCapsule.h>
#import <SpaceKit/SKRocket.h>

@interface SKMission NSObject

- (instancetype)initWithName:(NSString *)name 
                  launchDate:(NSDate *) launchDate 
                      rocket:(NSString *)rocket
                     capsule:(NSString *) capsule;
- (instancetype)initWithContentsofURL:(NSURL*)url
                                error:(NSError**)error; 
@property (copy) NSString *name;
@property (strong) NSDate *launchDate;
@property (copy) NSString *rocket;
@property (copy) NSString *capsule;
@property (copy) NSArray *crew;

///\returns \c YES if saved; \c NO with non-nil \c *error if failed to save;
///         \c NO with nil \c *error' if nothing needed to be saved.
- (BOOL) saveToURL:(NSURL *)url 
             error: (NSError **) error;
```

To:

```swift
open class SKMission: NSObject {
  public init!(name: String!, launchDate: Date!, rocket: String!, capsule: String!)
  public init (contentsOf url: URL!) throws

  open var name: String! { get } 
  open var launchDate: Date! { get }
  open var rocket: String! { get }
  open var capsule: String! { get }
  open var crew: [Any]! { get } 

  /// \returns \c YES if saved; \c NO with non-nil \c *error if failed to save; \c NO with 
  /// nil \c *error if nothing needed to be saved. 
  open func save(to url: URL!) throws 

  open func previousMissionsFlown(by astronaut: SKAstronaut!) -> Set ‹AnyHashable>! 
}
```

There's room for improvements from the generated interface:

- the API has many implicitly unwrapped optionals
- The `Any` and `AnyHashable` types are vague
- the `throws` method will sometimes throw when it shouldn't
- some method names could also be more swifty
- and more

## Provide richer type information

### Describe nullability to control optionals

When Swift imports an Objective-C pointer type, by default, it marks it as an implicitly unwrapped optional to tell you that this value could be `nil`.

#### Methods and Properties

Objective-C provides three nullability annotations, `nonnull`, `nullable`, `null_unspecified`, which let you say whether `nil` is a sensible value for a particular property, method parameter or method result:

```objc
// SKMission.h

#import <Foundation/Foundation.h>

@interface SKMission : NSObject

@property (readonly, nullable) NSString *name;

- (nonnull instancetype)initWithName:(nullable NSString *)name;

@end
```

> Objective-C doesn't enforce these annotations. They just document your intent.

- `nonnull` will be imported as a non-optional type in Swift
- `nullable` will be imported as an optional type in Swift
- `null_unspecified` will be imported as an implicitly unwrapped optional

Lastly, add the `NS_ASSUME_NONNULL_BEGIN` macro at the top of the header file and the matching end macro at the bottom, then delete all the `nonnull`s between them.

This is a convenience macro to save you typing `nonnull` in your headers.

```objc
// SKMission.h

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface SKMission : NSObject

@property (readonly, nullable) NSString *name;

- (instancetype)initWithName:(nullable NSString *)name;

@end

NS_ASSUME_NONNULL_END
```

#### Any Pointer

For other pointers that are not methods and properties, use `_Nonnull`, `_Nullable`, and `_Null_unspecified`.

```objc
// Misc.h

#import <Foundation/Foundation.h>

NSString * _Nonnull const SKRocketSaturnV;

@interface ResourceValueContainer : NSObject

- (BOOL)getResourceValue:(id _Nullable * _Nonnull)outValue error:(NSError**)error;

@end
```

#### Nullability mistakes

What happens when Objective-C returns `nil` for a value Swift thinks can't be optional? 

- If it's an `NSString` or `NSArray` on the Objective-C side, you get an empty Swift string or array
- If it's an Objective-C object, you might not even notice because Objective-C method calls ignore nils. But in some cases, you'll crash with a null pointer dereference or get other unexpected behavior

### Use Objective-C generics for Foundation types

```objc
// SKMission.h

#import <Foundation/Foundation.h>
#import <SpaceKit/SKAstronaut.h>

NS_ASSUME_NONNULL_BEGIN

@interface SKMission : NSObject

@property (readonly) NSArray<SKAstronaut *> *crew;

@end

NS_ASSUME_NONNULL_END
```

`NSArray<SKAstronaut *>` will translate into `[SKAstronaut]`

### Use `Int` for numbers 

In both Objective-C and Swift, it's conventional to use unsigned types (e.g. `UInt`, `uint8_t`, `UInt8`) when an integer represents a collection of bits and you want to perform bitwise operations on those bits.

The main reason people use `NSUInteger` in Objective-C is to indicate that a number's value is never negative. 

Objective-C enables this style with automatic conversions and carefully designed overflow behaviors, but these exact features can cause serious security bugs, so Swift doesn't include them. 

Instead, Swift requires you to explicitly convert unsigned types to signed if you wanted signed arithmetic, and stops execution if unsigned arithmetic would produce a negative result.

Apple's recommendation (and what they do in their frameworks): turn all `NSUIntegers` into `Int`s when Swift imports them.

### Strengthen stringly-typed constants

A `typedef` gets imported as a type-alias in Swift, and in both languages, that's just an exact synonym for the original type.

However, when we add the `NS_STRING_ENUM` macro after the `typedef`, this dramatically reshapes the Swift translation:  
it now imports as a struct with the constants nested inside it, making something that looks and feels just like an enum with a raw string value.

```objc
// SKRocket.h

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

typedef NSString *SKRocket NS_STRING_ENUM;

extern SKRocket const SKRocketAtlas;
extern SKRocket const SKRocketTitanII;
extern SKRocket const SKRocketSaturnIB;
extern SKRocket const SKRocketSaturnV;

NSInteger SKRocketStageCount(SKRocket);

NS_ASSUME_NONNULL_END
```

```swift
public struct SKRocket: RawRepresentable {
  public var rawValue: String
  public static let atlas: SKRocket
  public static let titanII: SKRocket
  public static let saturnIB: SKRocket
  public static let saturnV: SKRocket
}

public func SKRocketStageCount(_: SKRocket) -> Int
```

## Follow Objective-C conventions

- use `NS_DESIGNATED_INITIALIZER` for the main initializer of a class (other initializers will automatically get be tagged as `convenience` in Swift)
- use `NS_UNAVAILABLE` for initializers that you don't support (e.g. `NSObject`'s `init` when you don't want to override it and want people to use your initializers instead)

- use the Objective-C error convention correctly, use `NS_SWIFT_NOTHROW` otherwise

```objc
// SKMission.h
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface SKMission : NSObject

/// \returns \c YES if saved; \c NO with non-nil \c *error if failed to save;
///          \c NO with nil \c *error` if nothing needed to be saved.
- (BOOL)saveToURL:(NSURL *)url error:(NSError **)error NS_SWIFT_NOTHROW DEPRECATED_ATTRIBUTE;

/// @param[out] wasDirty If provided, set to \c YES if the file needed to be
///   saved or \c NO if there weren’t any changes to save.
- (BOOL)saveToURL:(NSURL *)url 
         wasDirty:(nullable BOOL *)wasDirty 
           error:(NSError **)error;

@end

NS_ASSUME_NONNULL_END
```

Will turn into:

```swift
Class SKMission: NSObject {
  @available(*, deprecated)
  public func save(to url: URL, error: AutoreleasingUnsafeMutablePointer..)

  public func save(to url: URL, wasDirty: UnsafeMutablePointer<ObjCBool>?) throws
}
```

- Use `NS_REFINED_FOR_SWIFT` when you re-define a method in Swift and want to hide the original objc implementation (when imported into Swift)
  - this macro adds two underscores to the beginning of the method's Swift name
  - When Xcode sees something with a leading underscore, it usually hides it from editor features like code completion and generated interfaces

## Address missing APIs

Swift can't import:

- C-style variadic parameters
- Flexible array members
- Forward declarations (like an `@class` or `@protocol` with a semicolon) that are never fully defined
- declarations involving un-importable types
- invalid redeclarations
- complicated macros

## Improve ergonomics in Swift

### Fix method names with `NS_SWIFT_NAME`

Instead of

```objc
- (NSSet<SKMission *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut;
```

that turns into:

```swift
func previousMissionsFlown(by astronaut: SKAstronaut) -> Set<SKMission>
```

Use `NS_SWIFT_NAME`:

```objc
- (NSSet<SKMission *> *)previousMissionsFlownByAstronaut:(SKAstronaut *)astronaut NS_SWIFT_NAME(previousMissions(flownBy:));
```

that becomes:

```swift
func previousMissions(flownBy astronaut: SKAstronaut) -> Set<SKMission>
```

`NS_SWIFT_NAME` is very powerful, for example it can be used to change a global function into a static/instance function of a type

### Error code enums

Use `NS_ERROR_ENUM` to convert an Objective-C enum and an error domain constant into a Swift enum conforming to `Error`:

```objc
//  SKError.h

#import <Foundation/Foundation.h>

extern NSString *const SKErrorDomain;

typedef NS_ERROR_ENUM(SKErrorDomain, SKErrorCode) {
    SKErrorLaunchAborted = 1,
    SKErrorLaunchOutOfRange,
    SKErrorRapidUnscheduledDisassembly,
    SKErrorNotGoingToSpaceToday
};
```

..will turn into:

```swift
public let SKErrorDomain: String

public struct SKError {
  public enum Code: Int {
    case launchAborted = 1
    case launchOutOfRange = 2
    case rapidUnscheduledDisassembly = 3
    case notGoingToSpaceToday = 4
  }

  public static var launchAborted: SKError.Code { get }
  public static var launchOutOfRange: SKError.Code { get }
  public static var rapidUnscheduledDisassembly: SKError.Code { get }
  public static var notGoingToSpaceToday: SKError.Code { get }

  public static var errorDomain: String { get }
}

extension SKError: Error {
  ...
}
```
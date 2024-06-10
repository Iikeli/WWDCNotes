# Expand on Swift macros

Discover how Swift macros can help you reduce boilerplate in your codebase and adopt complex features more easily. Learn how macros can analyze code, emit rich compiler errors to guide developers towards correct usage, and generate new code that is automatically incorporated back into your project. We’ll also take you through important concepts like macro roles, compiler plugins, and syntax trees.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10167", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Becca Royal-Gordon

## Why macros?

Swift has "derived protocol conformance" for many protocol like `CaseIterable`, `Codable`, `Hashable` and `Equatable`.

Macros help taking this to other places:
- Eliminate boilerplate
- Make tedious things easy
- Share with other developers in packages

## Design philosophy

Swift macros are very different than macros in C and Objective-C. They had 4 goals in mind when implementing macro support.

### First goal: Distinctive use sites

> The first goal is that it should be pretty obvious when you're using a macro.

> ...if you don't see #’s or @’s, you can be confident that there aren't any macros involved.

There are two kinds of macros.

#### Freestanding macros

```swift
return #unwrap(icon, message: "should be in the app bundle")
```

- Take the place of an expression or declaration
- Start with a # sign

#### Attached macros

```swift
@AddCompletionHandler func sendRequest() async throws -> Response
```

- Attached to another declaration
- Start with an @ sign

### Second goal: Complete, type-checked, validated

> You can't pass "1 +” to a macro because arguments have to be complete expressions. Nor can you pass an argument with the wrong type because macro arguments and results are type-checked, just like function arguments. And a macro's implementation can validate its inputs and emit compiler warnings or errors if something's wrong, so it's easier to be certain that you're using a macro correctly.

### Third goal: Inserted in predictable ways

> A macro can only add to the visible code in your program. It can't remove it or change it.

Even if you don't know what `#someUnknownMacro()` does, it can't delete the call to `finishDoingThingy()`.

```swift
func doThingy() {
  startDoingThingy()
  #someUnknownMacro()
  finishDoingThingy()
}
```

### Fourth goal: Macros are not magic

> Macros just add more code to your program, and that's something you can see right in Xcode.
>
> You can right-click on a macro's use site and ask to see what it expands into. You can set breakpoints in the expansion or step into it with the debugger. When the code inside a macro expansion doesn't compile, you'll see both where the error is in the expansion, and where that expansion goes in your source code. And all of these tools work even if the macro is provided by a closed-source library. 

## Translation model

> When Swift sees you call a macro in your code, like the "stringify" macro from the Xcode macro package template, it extracts that use from the code and sends it to a special compiler plug-in that contains the implementation for that macro.
> 
> The plug-in runs as a separate process in a secure sandbox, and it contains custom Swift code written by the macro's author. It processes the macro use and returns an "expansion," a new fragment of code created by the macro. The Swift compiler then adds that expansion to your program and compiles your code and the expansion together

```swift
/// Creates a tuple containing both the result of `expr` and its source code represented as a `String`
@freestanding(expression)
macro stringify<T>(_ expr: T) -> (T, String)
```

## Macro roles

Macro roles determines:
- Where it can be used
- What types of code it expands into
- Where the expansions are inserted

| Declaration | Behavior | Protocol |
|---|---|---|
| @freestanding(expression) | Creates a piece of code that returns a value | `ExpressionMacro` |
| @freestanding(declaration) | Creates one or more declarations | `DeclarationMacro` |
| @attached(peer) | Adds new declarations alongside the declaration it's applied to | `PeerMacro` |
| @attached(accessor) | Adds accessors to a property | `AccessorMacro` |
| @attached(memberAttribute) | Adds attributes to the declarations in the type/extension it's applied to | `MemberAttributeMacro` |
| @attached(member) | Adds new declarations inside the type/extension it's applied to | `MemberMacro` |
| @attached(conformance) | Adds conformances to the type/extension it's applied to | `ConformanceMacro` |

### Role composition

- A macro may have multiple attached roles
- Swift will expand all roles applicable where the macro was used
- At least one role must be applicable

## Macro implementation

Here the macro is defined to a external macro in another module:

```swift
/// Creates a tuple containing both the result of `expr` and its source code represented as a
/// `String`.
@freestanding(expression)
macro stringify<T>(_ expr: T) -> (T, String) = #externalMacro(
                                                   module: "MyLibMacros",
                                                   type: "StringifyMacro"
                                               )
```

A macro is declared like this:
```swift
import SwiftSyntax
import SwiftSyntaxMacros
import SwiftSyntaxBuilder

struct DictionaryStorageMacro: MemberMacro {
    static func expansion(
        of attribute: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        return [
           "init(dictionary: [String: Any]) { self.dictionary = dictionary }",
           "var dictionary: [String: Any]"
        ]
    }
}
```

When writing macros, you need to have some knowledge about SwiftSyntax and how to work with it.

*See more in the session "Write Swift Macros" and in the [SwiftSyntax package's documentation](https://swiftpackageindex.com/apple/swift-syntax/508.0.1/documentation/swiftsyntax).*

> Each of the expansion methods returns SwiftSyntax nodes that are inserted into the source code. A member macro expands into a list of declarations to add as members to the type, so the expansion method for a member macro returns an array of `DeclSyntax` nodes.

### Handling errors

To output compile errors to the developer, use the 

### Building syntax trees

- SwiftSyntax nodes are immutable, but they have lots of APIs that either create new nodes or return modified versions of existing nodes
- The SwiftSyntaxBuilder library adds SwiftUI-style syntax builders where some of the child nodes are specified by a trailing closure
- The string literal feature used in the DictionaryStorage property and initializer also supports interpolations

```swift
static func makeGuardStmt(wrapped: TokenSyntax,
                           originalWrapped: ExprSyntax,
                           message: ExprSyntax,
                           in context: some MacroExpansionContext) -> StmtSyntax {
    let messagePrefix = "Unexpectedly found nil: ‘\(originalWrapped.description)’ "
    let originalLoc = context.location(of: originalWrapped)!

    return """
        guard let \(wrapped) else {
            preconditionFailure(
                \(literal: messagePrefix) + \(message),
                file: \(originalLoc.file),
                line: \(originalLoc.line)
            )
        }
    """
}
```

## Writing correct macros

### Variable names

Swift macros don't prevent name conflicts, because sometimes you want to access names from outside your macro.

> ...sometimes, you even want to introduce a whole new name that non-macro code can access. Peer macros, member macros, and declaration macros basically exist entirely to do this. But when they do, they need to declare the names they're adding, so the compiler knows about them. And they do that inside their role attribute.

```swift
@attached(conformance)
@attached(member, names: named(dictionary), named(init(dictionary:)))
@attached(memberAttribute)
@attached(accessor)
macro DictionaryStorage(key: String? = nil)

@attached(peer, names: overloaded)
macro AddCompletionHandler(parameterName: String = "completionHandler")

@freestanding(declaration, names: arbitrary)
macro makeArrayND(n: Int)
```

#### Name specifiers

| Name | Description |
|---|---|
| overloaded | Creates a declaration with the same base name as the declaration it's attached to (attached only) |
| prefixed(<some prefix>) | Creates a declaration whose base name is <some prefix> followed by the base name of the declaration it's attached to; prefix can start with $ (attached only) |
| suffixed(<some suffix>) | Creates a declaration whose base name is the base name of the declaration it's attached to followed by <some suffix> (attached only) |
| named(<some name>) | Creates a declaration with the base name <some name> |
| arbitrary | Creates a declaration whose name cannot be described by any of the rules above |

### Don't use outside information
  
- Only use the information the compiler provides
  - Otherwise, tools won't know when they need to re-expand a macro
- Macros don't have access to file systems and network
- Sandbox can't stop you, but you still shouldn't:
  - Insert API results like current time, process ID, or random numbers
  - Save information in global variables between expansions

## Testing your macros
  
- Test your macro implementations to see if they expand as you expect
- Use standard tools like XCTest

```swift
import MyLibMacros
import XCTest
import SwiftSyntaxMacrosTestSupport

final class MyLibTests: XCTestCase {
    func testMacro() {
        assertMacroExpansion(
            """
            @DictionaryStorage var name: String
            """,
            expandedSource: """
            var name: String {
                get { dictionary["name"]! as! String }
                set { dictionary["name"] = newValue }
            }
            """,
            macros: ["DictionaryStorage": DictionaryStorageMacro.self])
    }
}
```

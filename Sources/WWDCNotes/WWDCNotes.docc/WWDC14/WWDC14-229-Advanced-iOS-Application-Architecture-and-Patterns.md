# Advanced iOS Application Architecture and Patterns

Explore a selection of high-level software engineering techniques presented in the context of Cocoa Touch. Learn how to manage complexity in large codebases by clearly defining where truth resides, by controlling state with Swift's powerful value types and immutability, and by thinking in terms of composition.

@Metadata {
   @TitleHeading("WWDC14")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc14/229", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Design information flow

Truth vs. derived values: 

- Where is truth? 
- Who really knows that state which one place should be consulted for it?
- Values derived from truth are in many ways like a cache and needed to be treated as such

## Define clear responsibilities

- Think of the inputs and outputs
- Separate out responsibilities from the rest of the program
- Use composition to build up larger pieces from smaller ones

Example: validators.

```swift
protocol Validator {
  validateWithError(error: NSErrorPointer) -> Bool
}
```

- Definition of “input” left open to interpretation

For example: 

```swift
class SignUpValidator: Validator {
  let usernameValidator = UsernameValidator()
  let setPasswordValidator = SetPasswordValidator()
  let emailAddressValidator = EmailAddressValidator()
  ...
}
```

## Simplify with immutability

Use Swift structs to avoid having objects changing data they're not supposed to change.
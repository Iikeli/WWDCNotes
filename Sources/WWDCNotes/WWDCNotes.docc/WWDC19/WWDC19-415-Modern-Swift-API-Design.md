# Modern Swift API Design

Every programming language has a set of conventions that people come to expect. Learn about the patterns that are common to Swift API design, with examples from new APIs like SwiftUI, Combine, and RealityKit. Whether you're developing an app as part of a team, or you're publishing a library for others to use, find out how to use new features of Swift to ensure clarity and correct use of your APIs.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/415", purpose: link, label: "Watch Video (41 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



The most important goal as an API designer: _Clarity at the point of use_.

## `@DynamicMemberLookup`

It can be used to expose all the properties of a hidden object as if they belonged to the main object.

```swift
@dynamicMemberLookup
struct Material { 
  public var roughness: Float
  public var color: Color 

  private var _texture: Texture 

  public subscript<T>(
  	dynamicMember keyPath: ReferenceWritableKeyPath<Texture, T>
  ) -> T { 
  	get { _texture[keyPath: keyPath] }
  	set { 
      if !isKnownUniquelyReferenced(&_texture) { 
      	_texture = Texture (copying: _texture) 
      }
      _texture[keyPath: keyPath] = newValue 
    }
  }
}
```

## `@PropertyWrapper`

- Provides similar benefits to the built-in lazy
- Eliminates boilerplate.
- Documents semantics at the point of definition.

Property Wrappers can be seen as generic getters and setters:

```swift
// Implementing a Property Wrapper 

@propertyWrapper 
public struct LateInitialized<Value> { 
	private var storage: Value?

  public init() { 
    storage = nil 
  }

  public var value: Value {
  	get { 
      guard let value = storage else {
      	fatalError("value has not yet been set!") 
      } 
      return value 
    }
    set { 
      storage = newValue 
    }
  }
}
```

```swift
// Uses of property wrappers expand into a stored property and a computed property public 

public struct MyType {
  @LateInitialized public var text: String 
}

// Compiler-synthesized code 👇🏻
public struct MyType {
  var $text: LateInitialized<String> = LateInitialized<String>()

  public var text: String { 
  	get { $text. value } 
  	set { $text. value = newValue }
  }
}
```

We can even pass values to the property initializer:

```swift
// Defensive Copying 

@propertyWrapper 
public struct DefensiveCopying<Value: NSCopying> {
	private var storage: Value 

  public init(initialValue value: Value) {
	  storage = value. copy () as! Value 
  }

  public var value: Value {
  	get { storage } 
  	set { 
      storage = newValue.copy() as! Value 
    }
  }
}
```

```swift
// Initializing the backing storage property: 

public struct MyType { 
  @DefensiveCopying(withoutCopying: UIBezierPath())
  public var path: UIBezierPath 
}
```

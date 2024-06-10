# Simplify C++ templates with concepts

Discover how C++20 features can take your C++ code to the next level. We’ll introduce concepts and explore how you can use it to find errors faster in your generic C++ code. We’ll also discuss the latest enhancements to the constexpr feature and show how you can leverage it to improve your app's performance by evaluating code at compile time.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110367", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(dagronf)
   }
}



Speaker: Alex Lorenz, Developer Tools Engineer

## C++ Templates

Prior to C++20, C++ programmers did not have a good way to specify template requirements when writing generic C++ code.

For example

```cpp
template<class T>
bool isOdd(T x) {
   return (x % 2) == 1;
}
```

While this looks good, the requirements for what types are allowed into 'isOdd' are not specified explicitly. So...

```cpp
// Passing a double into a function that implies that 'T' specialization is an integer type
assert(isOdd(1.1));
```

... causes a compiler error ("Invalid operands to binary expression (`double` and `int`)") in the **`isOdd`** function, which can be very hard to track down _why_.

## Using concepts

This is where **C++20 concepts** comes in. You can use concepts to validate template requirements in your generic C++ code.

C++20 allows us to replace the `class` specifier in the template definition with a **concept** to restrict the set of types that this template can be used with.

```cpp
#include <concepts>
template<std::integral T>
bool isOdd(T x) {
   return (x % 2) == 1;
}
```

The compiler will not even try to specialize this function template when T does not satisfy this concept and give us useful information as to why and where the _actual_ error occurred.

* Concepts are used to define the **intent** of your template code.
* The standard library provides a set of **core concepts** which are accessed through the `concepts` header file and validated during compile time.

### Built-in Concept Examples

| Concept | Description |
|----|----|
| `std::floating_point<double>` | Is 'double' a floating point? |
| `std::convertible_to<long, int>` | Can 'long' be converted to 'int' |
| `std::move_constructible<std::vector<int>>` | satisfied by types that can be constructed directly from another value of the same type |
| `std::equality_comparable`| Does the types have a valid '==' operator that works with a value of the same type. |

Other type examples :-

* Can a type be moved or copied?
* Check if a type is a callable object.

### The requires keyword

Instead of replacing `class` with a concept, you can use the `requires` keyword to restrict multiple concepts

```cpp
template<class T>
requires std::equality_comparable<T> && std::default_constructible<T>
bool isDefaultValue(const T &value) {
   return value == T();
}
```

## Create concepts

You can create your own concepts. These run at compile time, and are completely discarded after the validation occurs.

```cpp
template<class T>
Color computePixelColor(const T &shape, float x, float y) {
   float dist = shape.getDistanceFrom(x, y);
   if (dist <= 0.0) {
      return Color::white();
   }
   return Color::transparent();
}
```

### Create a 'shape' concept to validate the shape.

We create a concept that validates that the shape type has a `getDistanceFrom` function which takes two float parameters.

```cpp
template<class T>
concept Shape = requires(const T &shape) {
  shape.getDistanceFrom(0.0f, 0.0f)
}
```

* The argument list in the `requires` expression can declare values of any time. You can then use these values within the `requires` body.
* The body contains a set of requirements that must pass in order for the concept to be satisfied. **This is only needed at compile time to validate, and is discarded after validation.**

We can further extend this concept to check that the function returns the expected type.

```cpp
template<class T>
concept Shape = requires(const T &shape) {
  { shape.getDistanceFrom(0.0f, 0.0f) } -> std::same_as<float>;
}
```

So, this shape concept now validates :-

1. The `shape` has a `getDistanceFrom` function, which takes two float values.
2. The `getDistanceFrom` function returns a value which is the same as a float.

Now, I can create functions that only accept valid 'Shape' types

```cpp
template<Shape T>
Color computePixelColor(const T &shape, float x, float y) { … }
```

### Multiple variants of a function template

You can create multiple variants of function template using different concepts

```cpp
template<class T>
concept GradientShape = Shape<T> && requires(const T &shape) {
  { shape,getGradientColor(0,0f, 0,0f) } -> std::same_as<Color>;
}

template<GradientShape T>
Color computePixelColor(const T &shape, float x, float y) { … }
```

Thus, calling `computePixelColor` with a `GradientShape` object will call the correct function.

* The compiler will choose the **most specific** template function available, so even though a `GradientShape` works with both variants it will choose the most specific overload.


### Creating concepts

* Create concepts by identifying requirements in generic code
* Use `requires` expression(s) to validate behviour of types
* Use concepts to provide overloaded variants for generic functions


## Additional C++20 improvements in Xcode 14

### Compile time evaluation (constexpr)

* Reduce the cost of initialization for variables
* Verify constants at compile time

Mark your functions/declaractions `constexpr` if you want them to be run at compile time.

```cpp
constexpr Color Color::fromHexCode(std::string_view hexcode) { … }
constexpr const std::array<Color, 3> colorPalette = {
   Color::fromHexCode("#FF5733"),
   Color::fromHexCode("#5640FE"),
   Color::fromHexCode("#55FFCC")
};
```

This way, the `colorPalette` is guaranteed to be initialized with the contant values at compile time, significantly reducing startup time.

### New Supported C++20 Features

* std::bind_front
* Concepts
* Template Constraints
* Safe Mixed-Type
* Integral Comparisons 
* constexpr std::sort
* constexpr std::tuple 
* std::to_underlying 
* constexpr std::pair
* constexpr std::reverse
* constexpr std::swap
* constexpr Default Constructor for std::atomic
* using enum 
* Iterator Concepts
* Core Concepts Library
* Atomic Synchronization Library
* Contains Method For Associative Containers

### You should switch to C++20 today if you haven't already done so

* You can use the "C++ Language Dialect" setting in your Xcode project to upgrade to C++20
* C++20 does not require a minimum deployment target

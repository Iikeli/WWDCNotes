# Introducing Accelerate for Swift

Accelerate framework provides hundreds of computational functions that are highly optimized to the system architecture your device is running on. Learn how to access all of these powerful functions directly in Swift. Understand how the power of vector programming can deliver incredible performance to your iOS, macOS, tvOS, and watchOS apps.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/718", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What is Accelerate?

The primary purpose of Accelerate is to provide thousands of low-level math primitives that run on a CPU and support image and signal processing, vector arithmetic, linear algebra, and machine learning. Most of these primitives are hand tuned to the microarchitecture of the processor.

This improves performances and battery usage.

## Swift

Until now the Accelerate APIs were not really Swift-friendly, this year Apple has changed that particular in four libraries:

- **vDSP** (Digital Signal Processing): provides digital signal processing routines including arithmetic on large vectors, Fourier transforms, biquadratic filtering, and powerful type conversion.
- **vForce** (Basically vDSP with more geometry like `sin`/`cos`/`log`/`sqrt`): provides arithmetic and transcendental functions including trig and logarithmic routines. 
- **Quadrature** (historic name, it meant compute the area beneath a curve in a interval): dedicated to the numerical integration of functions.
- **vImage** (Rich collection of editing tools. Used along with CoreGraphics/CoreVideo): provides a huge selection of image processing functions and integrates easily with core graphics and core video.

Now using any of these libraries requires much less code in Swift, and it is also safer (in some APIs).

## Behind the Scenes

Accelerate gets its performance benefits by using vectorization.

Let’s make an example of multiplying two arrays: 

```swift
let a: [Float] = [10, 20, 30, 40] 
let b: [Float] = [1, 2, 3, 4] 
var c: [Float] = [0, 0, 0, 0] 

for i in 0 ..< c.count {
  c[i] = a[i] * b[i] 
}
```

By doing so, each pair of elements are separately loaded, multiplied together, and the results stored.
With Accelerate:

```swift
let a: [Float] = [10, 20, 30, 40] 
let b: [Float] = [1, 2, 3, 4] 
var c: [Float] = [0, 0, 0, 0] 

vDSP.multiply(a, b, result: &c) 
```

When using Accelerate, the calculation is performed on single instruction multiple data, or SIMD registers. 
These registers can perform the same instruction on multiple items of data by packing those multiple items into a single register. For example, a single 128-bit register can actually store four 32-bit floating point values. So, a vectorized multiply operation can simultaneously multiply four pairs of elements at a time. 
This means that not only will the task be quicker, it will also be significantly more energy efficient.

## Real Examples

Basically anything that operates on arrays (or matrices) with a `map`/`for-each` can use Accelerate to improve performances by at least x3/4.

Even by computing a sum, a conversion, etc.

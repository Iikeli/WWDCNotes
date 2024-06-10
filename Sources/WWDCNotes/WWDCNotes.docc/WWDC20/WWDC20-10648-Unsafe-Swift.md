# Unsafe Swift

What exactly makes code “unsafe”? Join the Swift team as we take a look at the programming language’s safety precautions — and when you might need to reach for unsafe operations. We’ll take a look at APIs that can cause unexpected states if not used correctly, and how you can write code more specifically to avoid undefined behavior. Learn how to work with C APIs that use pointers and the steps to take when you want to use Swift’s unsafe pointer APIs.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10648", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(dasautoooo)
   }
}


> To get the most out of this session, you should have some familiarity with Swift and the C programming language.
> 
> The following notes will assume you understand pointers in C.

## `Unsafe` and `Safe`
* The distinction between safe and unsafe constructs is the way their implementations deal with invalid input.
* Most operations in the standard library fully validate their input before executing.
* **Safe** operations have **well-defined** behavior on **all** input.
* **Unsafe** operations have **undefined** behavior on **some** input.
* Force unwrap operator is **safe** because we can fully describe its behavior for all possible inputs.
* Optional provides an **unsafe** force-unwrapping operation through its `unsafelyUnwrapped` property. This property does **not** verify the underlying value to be non-nil.
* By using `unsafe` properties, you assume full responsibility to fulfill its requirements.

```swift
// Optional's force unwrapping operator

let value: Int? = nil

print(value!) // Fatal error: Unexpectedly found nil while unwrapping an Optional value
```

```swift
// Unsafe force-unwrapping

let value: String? = "Hello"

print(value.unsafelyUnwrapped) // Hello
```

```swift
// Invalid use of unsafe force-unwrapping

let value: String? = nil

print(value.unsafelyUnwrapped) // Guaranteed fatal error only in debug builds
```

### Benefits of `unsafe` interfaces
* Interoperability with code written in C or Objective-C.
* Control over runtime performance.

### Safe code ≠ no crashes
* Safe APIs guarantee to stop execution by raising a fatal runtime error.
* Swift is a safe programming language means its language and library-level features fully validate their input.

## Unsafe Pointers
* Unsafe pointer types are roughly on the same level of abstraction as pointers in the C programming language.

> Treat it as a C pointer. Think about `malloc()`, `calloc()`, `free()`, how C treats an array and array arithmetic in C. 


### Memory
![][memory]
> This picture shows memory from lower memory address (top) to higher memory address (bottom).

* Swift has a flat memory model.
* A linear address space of individually addressable 8-bit bytes (64-bit).
* Memory address is a hexadecimal integer value.
* As app executes, the state of its memory keeps evolving.

### Manual memory management example

```swift
let ptr = UnsafeMutablePointer<Int>.allocate(capacity: 1)
ptr.initialize(to: 42)
print(ptr.pointee) // 42
ptr.deallocate()
ptr.pointee = 23 // UNDEFINED BEHAVIOR
```
* Allocating an `UnsafeMutablePointer<Int>` creates a storage location and gives back a pointer to it.
* Pointer gets invalidated as the underlying memory is deinitialized and deallocated.
* `ptr.pointee` is dereferencing pointer `ptr`.
* Dereferencing a `NULL` pointer is a serious programming error.
* Xcode provides Address Sanitizer to help you catch memory corruption errors.
* For more, refer to [Safely manage pointers in Swift](../10167/) session.

![][address_sanitizer]

### Mapping C pointers to Swift
A big reason to use pointers in Swift is interoperability with unsafe languages like C or Objective-C.
![][c_pointers_mapping]

```swift
// C:
void process_integers(const int *start, size_t count);

// Swift:
func process_integers(_ start: UnsafePointer<CInt>!, _ count: Int)
```

* C function processes a buffer of integer values.
* The `const int *start` gets translated into an implicitly unwrapped Optional unsafe pointer type in Swift.

## Use a pointer
```swift
let start = UnsafeMutablePointer<CInt>.allocate(capacity: 4)

start.initialize(to: 0)
(start + 1).initialize(to: 2)
(start + 2).initialize(to: 4)
(start + 3).initialize(to: 6)

process_integers(start, 4)

start.deinitialize(count: 4)
start.deallocate()
```

1. Use [`UnsafeMutablePointer`](https://developer.apple.com/documentation/swift/unsafemutablepointer) to allocate a dynamic buffer suitable for holding integer values. 
2. Use pointer arithmetic and dedicated initialization methods to set up the buffer's elements to particular values.
3. Call the C function, passing it the pointer to the initialized buffer.
4. Deinitialize and deallocate the buffer.

🚨 Every step is fundamentally **unsafe**:

* The lifetime of the allocated buffer is not managed by the return pointer.
* Remember to manually deallocate it at the appropriate time, or it will cause a memory leak.
* Initialization cannot automatically verify that the addressed location is allocated.
* Deinitialization only makes sense if the underlying memory has been previously initialized with values the correct type.

## [Buffer Pointers](https://developer.apple.com/documentation/swift/unsaferawbufferpointer/iterator)
Swift Standard Library provides four unsafe buffer pointer types:
`UnsafeBufferPointer<Element>`
`UnsafeMutableBufferPointer<Element>`
`UnsafeRawBufferPointer<Element>`
`UnsafeMutableRawBufferPointer <Element>`

* Buffer pointers check against out-of-bounds access through their subscript operation, which adds a little safety to it.

## Accessing contiguous collection storage
```swift
Sequence.withContiguousStorageIfAvailable(_:)
MutableCollection.withContiguousMutableStorageIfAvailable(_:)

String.withCString(_:)
String.withUTF8(_:)

Array.withUnsafeBytes(_:)
Array.withUnsafeBufferPointer(_:)
Array.withUnsafeMutableBytes(_:)
Array.withUnsafeMutableBufferPointer(_:)
```

## [Temporary pointers](https://developer.apple.com/documentation/swift/unsaferawbufferpointer/iterator)
Get a temporary pointer to an individual Swift value, then pass to C functions.

```swift
withUnsafePointer(to:_:)
withUnsafeMutablePointer(to:_:)
withUnsafeBytes(of:_:)
withUnsafeMutableBytes(of:_:)
```

* Generated pointers is only valid for the duration of the closure's execution. (Before it get deallocated from stack.)

```swift
// C:
void process_integers(const int *start, size_t count);

// Swift:
let values: [CInt] = [0, 2, 4, 6]

values.withUnsafeBufferPointer { buffer in
  print_integers(buffer.baseAddress!, buffer.count)
}
```

* Store input data in an Array value.
* Use `withUnsafeBufferPointer` method to temporarily get direct access to the array's underlying storage.
* Extract the start address and count values, pass them directly to the C function.

### Special syntax:

```swift
let values: [CInt] = [0, 2, 4, 6]

print_integers(values, values.count)
```

* Simply pass an array value to a function expecting an unsafe pointer.
* The compiler will automatically generate the equivalent `withUnsafeBufferPointer` for us.

### Implicit value-to-pointer conversions:
![][value_to_pointer]

## Example

Here is a C function provided by the Darwin module that we can use to query or update low-level information about the running system.

```swift
// C:
int sysctl(int *name, u_int namelen,
				 void *oldp, size_t *oldlenp,
				 void *newp, size_t *newlen);

// Swift:
func sysctl(
  _ name: UnsafeMutablePointer<CInt>!,
  _ namelen: CUnsignedInt,
  _ oldp: UnsafeMutableRawPointer!,
  _ oldlenp: UnsafeMutablePointer<Int>!,
  _ newp: UnsafeMutableRawPointer!,
  _ newlen: Int
) -> CInt
```

Create a function that retrieves the size of a cache line for the processor architecture we are running on.

```swift
import Darwin

func cachelineSize() -> Int {
	// The information is available under the identifier 'CACHELINE' in the hardware section.
    var query = [CTL_HW, HW_CACHELINE]
    // The information we want to retrieve is a C integer value.
    var result: CInt = 0
    // The size of integer type buffer.
    var resultSize = MemoryLayout<CInt>.size
    
    let r = sysctl(&query, CUnsignedInt(query.count), &result, &resultSize, nil, 0)
    
    // Sysctl is documented to return zero value on success.
    precondition(r == 0, "Cannot query cache line size")
    // Expect the call to set as many bytes as there are in a C integer value.
    precondition(resultSize == MemoryLayout<CInt>.size)
    
    return Int(result)
}

print(cachelineSize()) // 64
```

* The function will set `resultSize` to the number of bytes it copied into `result`.
* Because we only want to retrieve the current value, not set it, we supply nil value for the `new value` buffer, and set its size to zero.

Expand the code above into explicit closure based calls:

```swift
import Darwin

func cachelineSize() -> Int {
    var query = [CTL_HW, HW_CACHELINE]
    return query.withUnsafeMutableBufferPointer { buffer in
        var result: CInt = 0
        withUnsafeMutablePointer(to: &result) { resultptr in
            var resultSize = MemoryLayout<CInt>.size
            let r = withUnsafeMutablePointer(to: &resultSize) { sizeptr in
                sysctl(buffer.baseAddress, CUnsignedInt(buffer.count),
                       resultptr, sizeptr,
                       nil, 0)
            }
            precondition(r == 0, "Cannot query cache line size")
            precondition(resultSize == MemoryLayout<CInt>.size)
        }
        return Int(result)
    }
}

print(cachelineSize()) // 64
```
This code is functionally equivalent to the code above.

## Clousure-based vs. implicit pointers

```swift
var value = 42
withUnsafeMutablePointer(to: &value) { p in
  p.pointee += 1
}
print(value)  // 43
```

* Closure-based design makes the actual lifetime of the resulting pointer far more explicit.
* Helping us avoid lifetime issues.

```swift
var value2 = 42
let p = UnsafeMutablePointer(&value2) // BROKEN -- dangling pointer!
p.pointee += 1
print(value2)
```

* Passing a temporary pointer to the mutable pointer initializer escapes its value out of the initializer call.
* Accessing the resulting dangling pointer value is undefined behavior.

**Prefer to use closure-based APIs in pure Swift code.**

## Initializing contiguous collection storage
New initializers that allow us to create an `Array` or a `String` value by directly copying data into their underlying uninitalized storage.

```swift
Array.init(unsafeUninitializedCapacity:initializingWith:)
String.init(unsafeUninitializedCapacity:initializingUTF8With:)
```

### Example
Find out the kernel version of the operating system we're running on.

```swift
import Darwin

func kernelVersion() -> String {
	// Identified by VERSION entry in the kernel section.
    var query = [CTL_KERN, KERN_VERSION]
    var length = 0
    let r = sysctl(&query, 2, nil, &length, nil, 0)
    precondition(r == 0, "Error retrieving kern.version")
    
    // The initializer gives us a buffer point that we can pass through the sysctl function.
    return String(unsafeUninitializedCapacity: length) { buffer in
        var length = buffer.count
        // The function will copy the version string directly into this buffer.
        let r = sysctl(&query, 2, buffer.baseAddress, &length, nil, 0)
        precondition(r == 0, "Error retrieving kern.version")
        precondition(length > 0 && length <= buffer.count)
        // Check the last byte is zero, corresponding to the NUL character terminating a C string.
        precondition(buffer[length - 1] == 0)
        // Discard the NUL character.
        return length - 1
    }
}

print(kernelVersion())
// Darwin Kernel Version 19.5.0: Thu Apr 30 18:25:59 PDT 2020; root:xnu-6153.121.1~7/RELEASE_X86_64
```

* We don't know the size of the version string in advance, so we need to cause this control twice.
* On the first `sysctl` return, the `length` variable will get set to the number of bytes required to store the string.
* Don't need for manual memory management, by using this new String initializer.
* Get direct access to a buffer that will eventually become storage for a regular Swift string instance.
* Don't need to manually allocate or deallocate memory.

## Summary
* Follow the requirements of each unsafe interface.
* Keep unsafe API usage to the minimum.
* Use `UnsafeBufferPointer` for memory buffers.
* Test with the sanitizers.

[memory]: WWDC20-10648-memory
[address_sanitizer]: WWDC20-10648-address_sanitizer
[c_pointers_mapping]: WWDC20-10648-c_pointers_mapping
[value_to_pointer]: WWDC20-10648-value_to_pointer

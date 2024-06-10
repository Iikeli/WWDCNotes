# Safely manage pointers in Swift

Come with us as we delve into unsafe pointer types in Swift. Discover the requirements for each type and how to use it correctly. We’ll discuss typed pointers, drop down to raw pointers, and finally circumvent pointer type safety entirely by binding memory.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10167", purpose: link, label: "Watch Video (27 min)")

   @Contributors {
      @GitHubUser(dasautoooo)
      @GitHubUser(skhillon)
   }
}



> To get the most out of it, you should be familiar with Swift and the C programming language.

> You should at least have these pre-knowledge to understand this session: C programming language, pointer and array arithmetic in C, data representation in bit-level and run-time memory.

> I recommend you to read the note and watch this session again.

> This topic is not the kind of details that app developers typically need to worry about.

Managing pointers safely means knowing all the different ways they can be unsafe.

## Levels of safety

![][levels_of_safety]

| Level of safety | Swift API                      | Description                                                                                                              |
|-----------------|--------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| Safe            | `collections`, `slices`, `iterators` | Safe code. It's recommended write code at the highest safety level possible. **Not using pointers at all is a great strategy for code safety.**                                             |
| Unsafe          | `UnsafePointer<T>`               | UnsafePointer lets use pointers without worrying about type safety. |
| Raw             | `UnsafeRawPointer`               | UnsafeRawPointer lets you work with raw memory as a sequence of bytes.                                                   |
| Mutable type    | Memory-binding APIs            | Swift provides a few APIs for binding memory to types.                                                                     You're taking all the responsibility for pointer type safety. |

### Safe Swift code

![][safe_swift_code]

* Safe code isn't necessarily correct code but it does behave predictably.
* The compiler will catch the error if a programming error can lead to unpredictable behavior.
* Runtime checks guarantee the error will make the program crashes immediately.
* Safe code is really about error enforcement.

### Unsafe Swift code

![][unsafe_swift_code]

* Testing provides helpful diagnostics, but depends on the level of safety.
* Unsafe standard library APIs have assertions and debug builds that catch certain kinds of invalid input.
* Adding own preconditions to verify unsafe assumptions is a good practice.
* Sanitizer Diagnostics are great to pinpoint bugs, but don't catch all undefined behavior.
* When errors are not uncovered during testing they can lead to unexpected runtime behavior, the worst thing is corrupting or losing data.

## Pointer Safety
### Pointer's unsafe reason 1

![][pointer_lifetime_1]
![][pointer_lifetime_2]

* It needs a stable memory location before creating a pointer.
* The stable memory location has a limited lifetime - the memory location might not in the current stack frame or the memory gets deallocated directly.
* Any behavior is undefined, if a pointer accesses an invalid memory address.

### Pointer's unsafe reason 2

![][pointer_object_boundary]

* A memory location can fit in multiple objects. For example, a 64-bit memory can fit in two `Int32` type objects.
* Pointers are allowed to move to different memory addresses by adding offsets to the pointer.
* Adding or subtracting too large to the pointer, might access to the different object. 
* Accessing a pointer that has exceeded its object's boundary is undefined.

### Pointer's unsafe reason 3

![][pointer_type_1]
![][pointer_type_2]

* Pointers have their own types, they're different from the types of values in memory.
* If the pointer we have is a type of `Int16`, then we overwrite the memory location to store a `Int32` object, the pointer type will be inconsistent.
* Accessing the old pointer of type `Int16` is undefined behavior.

#### Pointer type bugs
* Different versions of complier can cause different program behavior.
* May cause unexpected behavior.
* May remain hidden for a long time.
* May be exposed at surprising times:
	* By safe-looking source change.
	* By a compiler update.

## Swift type-safe pointers
### Pointer type rules for Swift and C

* C has rules for "strict aliasing" and "type punning".
* Swift pointers can be used safely without knowing C rules.
* Swift pointers safely interoperate with C because they are, at minimum, as safe as C pointers. 
* In exchange, you need to take responsibility for object lifetime and object boundaries. 
* You can learn more in the [Unsafe Swift](https://github.com/skhillon/WWDC-Notes/blob/safe-pointers/content/notes/10648) talk.

### `UnsafePointer<T>` is a *typed pointer*

![][type_safe_pointer]

* In C, it's common to cast pointers to different types with both pointers continuing to refer to the same memory.
* `UnsafePointer<T>` only reads values of that type from memory.
* `UnsafeMutablePointer<T>` only reads or writes values of that type.
* It's undefined behavior in Swift to access a pointer whose type parameter does not match its memory location's bound type.
* Pointer types are enforced at compile time by Swift's type system.

### Pointers to variables

![][pointer_to_variables]

* Declare a variable of type int, then ask for a pointer, will get back a pointer to int.

### Pointers to arrays

![][pointer_to_arrays]

* Array storage is bound to the array element type.
* Asking for a pointer into array storage gives back a pointer to the arrays element type.

### Type-safe direct memory allocation

```swift
func directAllocation<T>(t: T, count: Int) {
    let tPtr = UnsafeMutablePointer<T>.allocate(capacity: count)
    tPtr.initialize(repeating: t, count: count)
    tPtr.assign(repeating: t, count: count)
    tPtr.deinitialize(count: count)
    tPtr.deallocate()
}
```

![][type_safe_memory_allocation]

* Allocate memory directly by calling the static allocate method on `UnsafeMutablePointer`.
* Allocation binds memory to its type parameter and returns a typed pointer to the new memory.
* Use the pointer to initialize memory only to the correct type. 
* In the initialized state, memory can be reassigned.
* De-initialize memory using the same typed pointer. Then can be safe to de-allocate.

### Composite types in memory

![][composite_type]

* Generally won't have two active pointers to the same memory location that disagree on the type.
* It's able to either get a pointer to the outer struct or a pointer to its property, they are both valid at the same time. 

## Swift *raw* pointers

* `UnsafeRawPointer` lets you refer to a sequence of bytes without specifying the type. 
* You take control over memory layout.

### Loading bytes with `UnsafeRawPointer`

![][loading_bytes_with_raw]

* It's able to interpret bytes as typed values. 
* It's always possible to cast from a typed pointer down to a raw pointer. 
* Operations on raw pointer only see the sequence of bytes in memory. 
* It's able to ask that raw pointer to load any type. 

#### Example

![][loading_bytes_with_raw_example]

* Call `.load(as: UInt32.self)` on a `Int64` pointer.
* It loads the lower 4 bytes from the memory location.
* Then it interprets the 4 bytes as a `UInt32` value.
* From a two's compliment number to an unsigned number.

### Storing bytes with `UnsafeMutableRawPointer`

![][storing_bytes_with_raw]

* Storing bytes is asymmetric with loading because it modifies the in-memory value. 
* Storing raw bytes does not de-initialize the previous value in memory. 
* To make sure the memory doesn't contain any object references.

#### Example

![][storing_bytes_with_raw_example]

* Call `.storeBytes(of: u, as: UInt32.self)` extracts 4 bytes from a `UInt32` value `u`, writing them into the upper 4 bytes of an in-memory `Int64` value.
* The typed pointer `iBytes` that already points to the in-memory value can still be used to access it, but with different value.
* Cannot cast a raw pointer back into a typed pointer because it conflicts with the memories bound type.
* In this case, `Int64` pointer overlaps with `UInt32` pointer.

### Raw pointers to variables

![][raw_pointer_to_variables]

* `UnsafeRawBufferPointer` is a collection of bytes, just like `UnsafeBufferPointer` is a collection of typed values.
* Buffer count is the size and bytes of the variables type.
* The collection index is a byte offset. (It's same as array arithmetic in C.)

### Raw pointers to mutable storage

![][raw_pointers_to_mutable]

* `withUnsafeMutableBytes` gives a collection of mutable bytes, so you can store `UInt` values and specific byte offsets.

### Raw pointers to arrays

![][raw_pointers_to_arrays]

* `withUnsafeBytes` method exposes the raw storage for the array elements.
* The buffer size is the array's count multiplied by the element stride.
* Some of those bytes could be padding for element alignment. (Data alignment)

### Raw pointers to `Data`

```swift
import Foundation

func readUInt32(data: Data) -> UInt32 {
    data.withUnsafeBytes { (buffer: UnsafeRawBufferPointer) in
        buffer.load(fromByteOffset: 4, as: UInt32.self)
    }
}

let data = Data(Array<UInt8>([0, 0, 0, 0, 1, 0, 0, 0]))
print(readUInt32(data: data))
```

* Foundation's `data` type is a collection of bytes. 
* `withUnsafeBytes` method exposes the underlying raw pointer for the duration of a closure. 
* Here we read the bytes start from offset 4, interprets as `UInt32`.

### Allocating raw storage

```swift
func rawAllocate<T>(t: T, numValues: Int) -> UnsafeMutablePointer<T> {
    let rawPtr = UnsafeMutableRawPointer.allocate(
            byteCount: MemoryLayout<T>.stride * numValues,
            alignment: MemoryLayout<T>.alignment)
    let tPtr = rawPtr.initializeMemory(as: T.self, repeating: t, count: numValues)
    // Must use the typed pointer ‘tPtr’ to deinitialize.
    return tPtr
}
```

![][allocate_raw_strorage]

* Using `UsafeMutableRawPointer.allocate` to directly allocate raw memory.
* Should compute the memory size and alignment in bytes.
* Memory state is neither initialized nor bound to a type after raw allocation.
* Specify the type of values to initialize memory. 
* `tPtr` is a typed pointer.
* Use typed pointer to de-initialize.
* There's no way to de-initialize with a raw pointer.
* The allocation doesn't care if memory is bound to a type or not.

#### Example: Contiguous storage for different types

```swift
func contiguousAllocate<Header>(header: Header, numValues: Int) -> (UnsafeMutablePointer<Header>, UnsafeMutablePointer<Int32>) {
    let offset = MemoryLayout<Header>.stride
    let byteCount = offset + MemoryLayout<Int32>.stride * numValues
    assert(MemoryLayout<Header>.alignment >= MemoryLayout<Int32>.alignment)
    let bufferPtr = UnsafeMutableRawPointer.allocate(
            byteCount: byteCount, alignment: MemoryLayout<Header>.alignment)
    let headerPtr = bufferPtr.initializeMemory(as: Header.self, repeating: header, count: 1)
    let elementPtr = (bufferPtr + offset).initializeMemory(as: Int32.self, repeating: 0, count: numValues)
    return (headerPtr, elementPtr)
}
```

![][allocate_raw_storage_example_1]

* We want to store unrelated types in the same contiguous block of memory. (As the image shows, we have type named `Header` and `Int32`.)
* `bufferPtr` is a raw pointer to a contiguous block of bytes. (It points to the first byte of the memory space.)

![][allocate_raw_storage_example_2]

* Initialize first few bytes of memory to the type of `Header`.

![][allocate_raw_storage_example_3]

* Initialize the remaining bytes to `Int32`.

This storage allocation technique is great for implementing standard library types like `set` and `dictionary`.

In general raw pointers are a kind of power tool that are good for implementing high-performance data structures, but we don't want to expose them too much.

#### Use case: decoding byte buffers
The more likely case where you want to use a raw pointer is when you have a buffer of bytes that's externally generated, and you want to decode those bytes into Swift types. 

![][raw_pointer_use_case]

* Read a descriptor to determine the sizes and types of subsequent data.
* Load the following data then decode to whatever type we want.


## Mutable type

* API names refer to the memory's 'bound type':
	* `assumingMemoryBound(to:)`
	* `bindMemory(to:capacity:)`
	* `withMemoryRebound(to:capacity:)`

* Can introduce undefined behavior or existing uses of typed pointers.
* Rule: every typed pointer access must agree with memory's bound type.

### `assumingMemoryBound(to:)`

#### Recovering a typed pointer

```swift
func takesIntPointer(_: UnsafePointer<Int>) { /* elided */ }

struct RawContainer {
    var rawPtr: UnsafeRawPointer
    var pointsToInt: Bool
}

func testContainer(numValues: Int) {
    let intPtr = UnsafeMutablePointer<Int>.allocate(capacity: numValues)
    let rc = RawContainer(rawPtr: intPtr, pointsToInt: true)
    // ...
    if rc.pointsToInt {
        takesIntPointer(rc.rawPtr.assumingMemoryBound(to: Int.self))
    }
}
```

* `RawContainer.rawPtr` holds raw memory.
* ⚠️ Use `assumingMemoryBound(to: T.self)` when memory is already bound to 'T' by a previous operation.

#### Pointing to tuple elements

```swift
func takesIntPointer(_: UnsafePointer<Int>) { /* elided */ }

func testPointingToTuple() {
    let tuple = (0, 1, 2)
    withUnsafePointer(to: tuple) { (tuplePtr: UnsafePointer<(Int, Int, Int)>) in
        takesIntPointer(UnsafeRawPointer(tuplePtr).assumingMemoryBound(to: Int.self))
    }
}
```

* `withUnsafePointer` gives back a pointer `tuplePtr` to the tuple type which is incompatible with `UnsafePointer<Int>` type.
* Memory bound to a tuple is also bound to its element types.
* Construct a raw pointer deliberately erasing the type of tuple pointer.
* Use `assumingMemoryBound` to create a pointer to `Int`. 
* Homogeneous tuples have guaranteed layout. (one value after another)

#### Pointing to struct properties

```swift
func takesIntPointer(_: UnsafePointer<Int>) { /* elided */ }

struct MyStruct {
    var status: Bool
    var value: Int
}

func testPointingToStructProperty() {
    let myStruct = MyStruct(status: true, value: 0)
    withUnsafePointer(to: myStruct) { (ptr: UnsafePointer<MyStruct>) in
        let rawValuePtr =
                (UnsafeRawPointer(ptr) + MemoryLayout<MyStruct>.offset(of: \MyStruct.value)!)
        takesIntPointer(rawValuePtr.assumingMemoryBound(to: Int.self))
    }
}
```

* `MyStruct` has an integer property. 
* `withUnsafePointer` gives a typed pointer to `myStruct`.
* By casting the struct pointer down to a raw pointer and adding that byte offset, we get a raw pointer to the value property.
* A property's memory is always bound to the properties declared type so it's safe to call `assumingMemoryBound` to to get a pointer to an `Int`.
* ⚠️ Struct layout is not guaranteed - rawValuePtr only points to a single pointee.

**Simple alternative way:**

```swift
func takesIntPointer(_: UnsafePointer<Int>) { /* elided */ }

struct MyStruct {
    var status: Bool
    var value: Int
}

let myStruct = MyStruct(status: true, value: 0)
    takesIntPointer(&myStruct.value)
}

```

### `bindMemory(to:capacity:)`
* `bindMemory` API lets you change memories bound type. 
* If the memory location was not already bound to a type, It just binds the type for the first time. 
* If the memory is already bound to a type then it rebinds the type. 

```swift
func testBindMemory() {
    let uint16Ptr = UnsafeMutablePointer<UInt16>.allocate(capacity: 2)
    uint16Ptr.initialize(repeating: 0, count: 2)
    let int32Ptr = UnsafeMutableRawPointer(uint16Ptr).bindMemory(to: Int32.self, capacity: 1)
    // Accessing uint16Ptr is now undefined
    int32Ptr.deallocate()
}
```

**Changing the bound type of a memory region:**

* Modifies the abstract memory state
* Reinterprets the memory region's raw bytes in place
* Invalidates existing typed pointers
* Can be undefined for variable, array, and collection storage
* Facilitates low-level implementation of Swift - note application code

### `withMemoryrebound(to:capacity)`
 
Temporarily changing the bound type.

```swift
func takesUInt8Pointer(_: UnsafePointer<UInt8>) { /* elided */ }

func testWithMemoryRebound(int8Ptr: UnsafePointer<Int8>, count: Int) {
    int8Ptr.withMemoryRebound(to: UInt8.self, capacity: count) {
        (uint8Ptr: UnsafePointer<UInt8>) in
        // int8Ptr cannot be used within this closure
        takesUInt8Pointer(uint8Ptr)
    }
    // uint8Ptr cannot be used outside this closure
}
```
* `withMemoryrebound(to:capacity)` gives a pointer that's guaranteed to be valid for the scope of its closure.

### Using `bindMemory(to:capacity:)` safely
* `withMemoryrebound(to:capacity)` limitations:
	* Requires a pointer to the original type
	* Both types require the same stride

* To call `bindMemory(to:capacity:)` directly, follow the same principles:
	* Limit pointer use to a controlled scope
	* Rebind memory back to the original type when the scope ends

### Memory-binding APIs

* `assumingMemoryBound(to:)`
	* Recover a type-erased pointer type
	* ⚠️ Requires prior knowledge of the memory's bound type state

* `bindMemory(to:capacity:)`
	* Global change to the memory's bound type state
	* ⚠️ Low-level operation that invalidates existing typed pointers

* `withMemoryrebound(to:capacity)`
	* Temporarily change memory's bound type state
	* ⚠️ Useful for calling C APIs that disagree on types

### Safely reinterpreting bytes
```swift
❌
let uint32Ptr = rawPtr.bindMemory(to: UInt32.self)
return uint32Ptr.pointee
```

* Call `bindMemory` to get a pointer of the type it wants to read.
* But in the process of creating that pointer we've changed memory state and probably invalidated other pointers.

```swift
✅
return rawPtr.load(as: UInt32.self)
```

* Avoids changing the in-memory type and invalidating other pointers
* Type-safe: only layout compatibility matters
* A typed pointer can be cast to a raw pointer
* `withUnsafeBytes` provides a raw buffer for variables, arrays, or Data objects

### Layering types on top of raw memory

Let's say you want to view a region of memory as a sequence of elements with a specific element type, but the underlying storage is exposed as a raw pointer and may be viewed as different types by different parts of the code.

You could easily create a wrapper around that raw pointer to preserve your element type. 

```swift
struct UnsafeBufferView<Element>: RandomAccessCollection {
    let rawBytes: UnsafeRawBufferPointer
    let count: Int

    init(reinterpret rawBytes: UnsafeRawBufferPointer, as: Element.Type) {
        self.rawBytes = rawBytes
        self.count = rawBytes.count / MemoryLayout<Element>.stride
        precondition(self.count * MemoryLayout<Element>.stride == rawBytes.count)
        precondition(Int(bitPattern: rawBytes.baseAddress).isMultiple(of: MemoryLayout<Element>.alignment))
    }

    var startIndex: Int { 0 }

    var endIndex: Int { count }

    subscript(index: Int) -> Element {
        rawBytes.load(fromByteOffset: index * MemoryLayout<Element>.stride, as: Element.self)
    }
}

func testBufferView() {
    let array = [0,1,2,3]
    array.withUnsafeBytes {
        let view = UnsafeBufferView(reinterpret: $0, as: UInt.self)
        for val in view {
            print(val)
        }
    }
}
```

## Summary
* Try to avoid using pointers
* Avoid using typed pointers to reinterpret memory as different types
* Use `UnsafeRawBufferPointer` to:
	* Reinterpret raw bytes as different types
	* Decode Swift types from a byte stream
	* Implement a container to hold different types in contiguous memory

[levels_of_safety]: WWDC20-10167-levels_of_safety
[safe_swift_code]: WWDC20-10167-safe_swift_code
[unsafe_swift_code]: WWDC20-10167-unsafe_swift_code
[pointer_lifetime_1]: WWDC20-10167-pointer_lifetime_1
[pointer_lifetime_2]: WWDC20-10167-pointer_lifetime_2
[pointer_object_boundary]: WWDC20-10167-pointer_object_boundary
[pointer_type_1]: WWDC20-10167-pointer_type_1
[pointer_type_2]: WWDC20-10167-pointer_type_2
[type_safe_pointer]: WWDC20-10167-type_safe_pointer
[pointer_to_variables]: WWDC20-10167-pointer_to_variables
[pointer_to_arrays]: WWDC20-10167-pointer_to_arrays
[type_safe_memory_allocation]: WWDC20-10167-type_safe_memory_allocation
[composite_type]: WWDC20-10167-composite_type
[loading_bytes_with_raw]: WWDC20-10167-loading_bytes_with_raw
[loading_bytes_with_raw_example]: WWDC20-10167-loading_bytes_with_raw_example
[storing_bytes_with_raw]: WWDC20-10167-storing_bytes_with_raw
[storing_bytes_with_raw_example]: WWDC20-10167-storing_bytes_with_raw_example
[raw_pointer_to_variables]: WWDC20-10167-raw_pointer_to_variables
[raw_pointers_to_mutable]: WWDC20-10167-raw_pointers_to_mutable
[raw_pointers_to_arrays]: WWDC20-10167-raw_pointers_to_arrays
[allocate_raw_strorage]: WWDC20-10167-allocate_raw_strorage
[allocate_raw_storage_example_1]: WWDC20-10167-allocate_raw_storage_example_1
[allocate_raw_storage_example_2]: WWDC20-10167-allocate_raw_storage_example_2
[allocate_raw_storage_example_3]: WWDC20-10167-allocate_raw_storage_example_3
[raw_pointer_use_case]: WWDC20-10167-raw_pointer_use_case

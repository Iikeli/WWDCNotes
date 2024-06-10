# Link fast: Improve build and launch times

Discover how to improve your app's build and runtime linking performance. We'll take you behind the scenes to learn more about linking, your options, and the latest updates that improve the link performance of your app.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110362", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What is linking?

A linker is needed whenever we'd like to use code written in libraries or frameworks

Two types of linking:

- Static linking
  - happens when you build your app
  - can impact both your app build time and app size

- Dynamic linking
  - happens when your app is launch
  - can impact your app launch time

## What is static linking?

Let's have a look at its history to explain what it is.

### 1970s

Initially programs were simple: just one source file. Building was running the compiler on your single source file and it produced the executable program.

However, this didn't scale: separating a program into multiple source files was great both for readability and also meant not re-compiling every function, every time you build.

To make multiple source file programs possible, the compiler was split into two parts: 

- the first (`cc`, C compiler) part compiles source code to a new intermediate "relocatable [object][Object_file]" <kbd>.o</kbd> file
- The second part (`ld`, static linker) reads the relocatable <kbd>.o</kbd> files and produces a single executable program

### Late 1970s - Static libraries

Sharing multiple <kbd>.o</kbd> files to share functionality across programs was cumbersome, so the concept was Static libraries was born.

At the time the standard way to bundle files together was with the archiving tool `ar`, used for backups and distributions.

A static library was just multiple <kbd>.o</kbd> files put into an archive via `ar`, resulting in a <kbd>.a</kbd> static library file.

`ld` was taught how to read <kbd>.o</kbd> files directly out of an archive file <kbd>.a</kbd>

This worked great, however now programs size started to grow the more static libraries were used:  
to solve this `ld` started pull <kbd>.o</kbd> files from a static library only when they'd resolve some undefined symbol. This is called selective loading.

Selective loading meant that each program would only get the <kbd>.o</kbd> objects of a <kbd>.a</kbd> library that the program actually needed.

## Recent [ld64][ld64] improvements

> [ld64][ld64] is Apple's static linker

- 2x faster in Xcode 14 thanks to better utilization of machine cores

Some of the ways `ld64` was improved:

- content copied in parallel from the input to the output file
- multiple `__LINKEDIT` sections built in parallel
- UUID computation and codesigning hashes done in parallel
- Optimized algorithms (e.g., exports-trie builder now uses C++ string_view objects to represent the string slices of each symbol)
- Accelerated UUID computation (now SHA256)
  - adoption of latest crypto libraries which take advantage of hardware acceleration when computing the UUID of a binary

### Static linking best practices

When to use static libraries?

- stable code is good
- consider moving code under active development out of a static library to reduce build time

#### Linker options

- Some options that can effect your app link time. 
- Add them in your target/product <kbd>Build Settings</kbd> tab, on <kbd>Other Linker Flags</kbd>

`-all_load` (`-force_load`)

- Selective loading from archives has slows down the linker, because, to make builds reproducible and follow traditional static library semantics, the linker has to process static libraries in a fixed, serial order
- If you don't need this behavior, pass `-all_load` to the linker
- this flag enables linker to parse all <kbd>.a</kbd> files in parallel, just like <kbd>.o</kbd> files
- May get duplicate symbol errors if multiple static libraries offer the same symbol:
  - if your app does clever tricks where it has multiple static libraries implementing the same symbols, and depends on the command line order of the static libraries to drive which implementation is used, then this option is not for you

- `-all_load` may make your program bigger because "unused" code is now being added in. To compensate for that, you can use the linker option `-dead_strip`, that will cause the linker to remove unreachable code and data

`-no_exported_symbols`

- one part of the `__LINKEDIT` segment that the linker generates is an exports trie, which is a prefix tree that encodes all the exported symbol names, addresses, and flags.
- whereas all dylibs need to have exported symbols, a main app binary usually does not need any exported symbols (because, usually nothing ever looks up symbols in the main executable)
- with this flag the linker skips the creation of the trie data structure in `__LINKEDIT`
- ⚠️ trie exports are required if:
  - the app loads plugins
  - you use XCTest with your app as the host environment to run XCTest bundles

To see how big is your trie (and check whether it's worth skipping its creation) run:

```shell
$ dyld_info -exports /path/to/binary | Wc -l
```

`-no_deduplicate`

- the linker combines C++ functions that have the exact same code but different name (happens a lot due to C++ template expansions)
  - this is an expensive algorithm
  - the linker only looks at weak-def symbols, which are the ones the C++ compiler emits for template expansions that were not inlined
  - this is an app size optimization

- Xcode adds this flag for Debug builds (builds faster, but app binary is slightly larger)
- Cland adds this flag if you run clang link line with `-O0`
- use `-no_deduplicate` in your custom Debug builds systems

## What is dyamic linking?

Let's start from its history to explain what it is.

### Early 1990s - Dynamic libraries

> (picking up from where we left with static linking's history)

The more static libraries a program uses: 

- the slower a program linking time is (at build time)
- the bigger a final executable size is

Instead of using `ar` for libraries (and output a <kbd>.a</kbd> file), we could `ld` for libraries (and output a <kbd>.dylib</kbd> file). These new libraries are called dynamic libraries ("dylibs"), also known as DSOs or DLLs on other platforms.

The difference now is that `ld` treats linking with a dynamic library differently:

- instead of copying code out of the library into the final program, the linker just records a kind of _promise_
- That is, it records the symbol name used from the dynamic library and what the library's path will be at runtime

Thanks to this:

- the linker no longer get copies of library code in your executable
- your program's static link time is now proportional to the size of your code, independent of the number of dylibs you link with
- your program executable is only effected by your code (and other static libraries, but not dynamic ones)

When executing, the Virtual Memory system will reuse the same loaded dynamic library across multiple processes (that need to use that <kbd>.dylib</kbd>)

Cons:

- slower launch time
  - launching the app is no longer just loading one executable, but also all the associated <kbd>.dylib</kbd> which then need to be linked

- more dirty memory (`__DATA` pages)
  - with static libraries, the linker would co-locate all globals from all static libraries into the same `__DATA` pages in the main executable
  - with dynamic libraries, each library defines its own `__DATA` page

- requires a runtime linker (dynamic linker)
  - `dyld` will need to fulfill the _promise_ made during build time, which means it need to resolve the _promised_ symbols to your executable

### Inside dynamic linking

- An executable binary is divided up into segments, such as `__TEXT`, `__DATA`, `__LINKEDIT`
- segments are always a multiple of the page size for the OS
- each segment has a different permission
  - `__TEXT`, segment that contains your code, has execute permissions: the CPU may treat the bytes on the page as machine code instructions

- At runtime, `dyld` has to [`mmap()`][mmap()] the executables into memory with each segments' permissions

All the above is true for <kbd>.dylib</kbd>s as well

### Fixups

- the main executable has various pointers to symbols belonging to the `.dylib`s
- those pointers (and other memory allocations) cannot be known until runtime
- because `__TEXT` segments cannot change (at least in system based on code signing), these dynamic symbols/call sites becomes a call to a stub synthesized by the linker in `__TEXT`
- the stub loads a pointer from the `__DATA` segment, and jumps to that location
- unlike `__TEXT`, the `__DATA` segment can change at runtime (when copied into memory)
- hence, when `dyld` is resolving our _promises_, really it's just setting the correct pointers into our executable `__DATA` segment (in memory) pointing to the relevant <kbd>.dylib</kbd> symbols (also loaded into memory)

The `__LINKEDIT` segment contains the information dyld needs to drive what fixups are done

Three kind of fixups:

- rebases
  - when a dylib or app has a pointer that points within itself
  - needed due to ASLR, meaning that even internal pointers need to be rebased at app launch

- binds
  - symbolic references
  - their target is a symbol name (not a number like in rebases, as those were just offsets of the targets within the image)
  - e.g., a pointer to the function `malloc`: 
    - the string `_malloc` is stored in `__LINKEDIT` 
    - `dyld` uses that string to look up the actual address of `malloc` in the exports trie of <kbd>libSystem.dylib</kbd>
    - Then, `dyld` stores that value in the location specified by the bind

- chained
  - new this year
  - makes `__LINKEDIT` smaller
  - instead of storing all the fixup locations, `__LINKEDIT` stores just where the first fixup location is in each `__DATA` page, as well as a list of the imported symbols
  - the rest of the info is encoded in `__DATA`
  - it's called chained because, in the 64-bit pointer location in `__DATA`, some of the bits contain the offset to the next fixup location (hence we jump from fixup to fixup in a chain-like manner)
  - supported when deploying to iOS 13.4 and later

### How dyld works

1. `dyld` starts with the main executable, and it parses the mach-o find the dependent dylibs (the _promised_ dynamic libraries your executable needs in order to run)
2. For each <kbd>.dylib</kbd>, `dyld` finds them and `mmap()`s them
3. then `dyld` does step 1-2 recursively for each <kbd>.dylib</kbd>
4. once everything is loaded, `dyld` looks up all the bind symbols needed
5. once the lookup is completed, `dyld` uses those addresses when applying fixups
6. once all the fixups are done, `dyld` runs initializers, bottom up

Since 2017, steps 1 to 4 are cached, as they're the same at every app launch (they need to be redone only on app/OS updates).

## Recent [dyld][dyld] improvements

> [dyld][dyld] is Apple's dynamic linker

### Page-in linking

- `dyld` no longer applies fixups to all dylibs at launch
- instead, the kernel applies fixups to your `__DATA` pages lazily, on page-in
  - it has always been the case that the first use of some address in some page of an `mmap()`ed region triggered the kernel to read in that page
  - now, if it is a `__DATA` page, the kernel will also apply the fixup that page needs

- Apple OSes had special case of page-in linking for over a decade for OS dylibs in the dyld shared cache.
  - this year page-in linking has been generalized and made it available to everyone

- reduces dirty memory
- reduces launch time
- `DATA_CONST` pages are clean, which means they can be evicted and recreated just like `__TEXT` pages, reducing memory pressure
- available in iOS 16, macOS 13, and watchOS 9
- requires chained fixups, thus requiring your app to target iOS 13.4 or later
  - because with chained fixups, most of the fixup information will be encoded in the `__DATA` segment on disk, which means it is available to the kernel during page-in

- does not work for `dlopen()`, just dylibs linked at launch (in this case `dyld` does the fixups during the `dlopen` call

## Dynamic linking best practices

- use fewer dylibs
- optimize or eliminate static initializers (code that always runs pre-`main`)
- find sweet spot for static vs. dynamic libraries

## New tools

Two new tools:

`dyld_usage`

- cli tool that logs dyld operations (similar to `fs_usage`)
- available in macOS 13
- uses same mechanism as Instruments.app to trace

`dyld_info`

- allows you to inspect binaries (similar to `nm` or `otool`)
- can show info about mach-o files and dylibs in the dyld cache (`dyld_info` tool uses the same code as dyld and can thus see files/binaries not on disk)
- available in macOS 12
- how to view the exports (will show all the exported symbols in the dylib, and the offset of each symbol from the start of the dylib): `$ dyld_info -exports /path/to/bin`
- how to view the fixups: `$ dyld_info -fixups /path/to/bin`

[ld64]: https://github.com/apple-opensource/ld64
[dyld]: https://github.com/apple-opensource/dyld
[Object_file]: https://en.wikipedia.org/wiki/Object_file
[mmap()]: https://en.wikipedia.org/wiki/Mmap
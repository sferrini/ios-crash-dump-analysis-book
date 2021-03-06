# Migration Notes

This document summarizes the portability issues with the current code base in respect to porting
to Apple Silicon.  Note, our Xcode will only build for macOS, not iOS/tvOS/watchOS.

## Existing Code

### `icdab_as`

The prologues for the assembly of egg.s was out of date.  To get a proper modern prologue, we obtain it with the `scaffold.c` file.
>  Product > Perform Action > Assemble `scaffold.c`

### `icdab_asl`

No changes needed.

### `icdab_cycle`

I checked and this works correctly - it has a retain cycle caught by memory logging via Malloc Stack.

### `icdab_edge`

We properly get address sanitizer picking up OvershootAllocated.  We see the 0xAA scribble with Malloc Scribble enabled.

### `icdab_nsdata`

No changes needed.

### `icdab_planets`

No changes needed.

### `icdab_sample`

No changes needed.

### `icdab_sema`

This has multiple targets, `icdab_sema_{mac, ios}`.  The mac variant compiles ok on the main Xcode 12.0 beta (12A8158a). It runs ok on Apple Silicon.

### `icdab_sema`

No changes needed.  But this app comes in two flavours, iOS and macOS so needs to be tested on Apple Silicon.

### `icdab_thread` 

I have migrated this to OS X running ARM64.  I used TargetConditionals.h  Unfortunately, we can no longer
`thread_set_state` from another process so we don't get the kind of crash dump we are looking for.  Our goal
is to get the crash dump to report that the number of calls to `thread_set_state` was non-zero from other processes.
Luckily we have historical data to show the statistic.  If we figure out a way forwards, we'll revisit this.

### `icdab_wrap`

No changes needed.

## New Code

### Fixed arg called with variadic args

Write an example program that calls a function implemented with fixed arguments, with variadic arguments on the call site, and show it crashing on ARM; see [Addressing Architectural Differences](https://developer.apple.com/documentation/apple_silicon/addressing_architectural_differences_in_your_macos_code)

I did not do this because it is in fact related to the more realistic use case below.

### Dynamic calls with `objc_msgSend`

Write an example program that unsafely is transformed to variadic by `objc_msgSend`; see [Addressing Architectural Differences](https://developer.apple.com/documentation/apple_silicon/addressing_architectural_differences_in_your_macos_code)

I wrote the app but could not cause it to crash via the realistic use case of objective c message runtime calling.

### Boolean from Int failure

Since 1024 cast to BOOL is false on x86 but true on arm64 write some code which leads to a logic crash; see [Addressing Architectural Differences](https://developer.apple.com/documentation/apple_silicon/addressing_architectural_differences_in_your_macos_code)

I can't think of a realistic use case that crashes with this portability issue.

### Execute data as code

Write a simple program that places assembly code in a data array and then jumps into it.  This simulates writing a JIT compiler.  The aim is to trigger execute on readonly data segments; see [Porting JIT on Apple Silicon](https://developer.apple.com/documentation/apple_silicon/porting_just-in-time_compilers_to_apple_silicon)

There is an interesting tutorial on JIT; https://github.com/spencertipping/jit-tutorial

I studied JIT and found a more interesting case of Failed Crashes as a result of the analysis.

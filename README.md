# arm_neon_clangd_fix
This repository is an attempt to easily fix a [clangd](https://github.com/clangd/clangd) bug that occurs when 
cross-compiling to arm with a GNU toolchain.

## The bug
This bug occurs because clangd actually uses clang under the hood, but tries to
parse GNU's version of the `arm_neon.h` header, which uses GNU intrisics clangd
doesn't kow about.

It is referenced in these two issues : [#1707](https://github.com/clangd/clangd/issues/1707) [#1753](https://github.com/clangd/clangd/issues/1753)

## The fix
The work-around I found is to force CLANGD to parse the `arm_neon.h` *llvm's* header,
while continuing to use the GNU one for the actual compiler.

To do so I created a copy of llvm's `arm_neon.h`, that defines the header guard definition
of the GNU's `arm_neon.h`. This way even when including the GNU version, clangd
won't see the GNU intrisics and therefore won't complain.

Technically this had been done by modifying the header gard from __ARM_NEON_H (llvm) to _AARCH64_NEON_H_ (GNU)

## How-to
1. Add a __CLANGD__ compile definition to clangd parsing, to differentiate clangd and not clangd mode. You can simply add this to your `.clangd` file :
```
---
CompileFlags:
  Add: [-D__CLANGD__]
```

2. Get the custom header. You can do it in various way but I provided a CMake 
configuration file compatible with FetchContent if you don't wanna pollute your
repository with ~85.000 lines of useless C code.
```cmake
if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
  include(FetchContent)
  FetchContent_Declare(
    arm_neon_clangd_fix
    GIT_REPOSITORY git@github.com:maxmarsc/arm_neon_clangd_fix.git
    GIT_TAG main
    GIT_SHALLOW ON
  )
  FetchContent_MakeAvailable(arm_neon_clangd_fix)

  link_libraries(arm_neon_clangd_fix)
endif()
```

3. Force clangd to include the header in files where neon intrisics will be used
```c
#if defined(__aarch64__) && defined(__CLANGD__)
#include "arm_neon_clangd_fix.h"
#endif
```
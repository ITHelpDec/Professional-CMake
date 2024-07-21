# How to build

```bash
cmake -S . -B build
```

...will produce the following...

```
-- The C compiler identification is AppleClang 15.0.0.15000309
-- The CXX compiler identification is AppleClang 15.0.0.15000309
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
top:           CMAKE_SOURCE_DIR         = <rootdir>/variables
top:           CMAKE_BINARY_DIR         = <rootdir>/variables/build
top:           CMAKE_CURRENT_SOURCE_DIR = <rootdir>/variables
top:           CMAKE_CURRENT_BINARY_DIR = <rootdir>/variables/build
src:           CMAKE_SOURCE_DIR         = <rootdir>/variables
src:           CMAKE_BINARY_DIR         = <rootdir>/variables/build
src:           CMAKE_CURRENT_SOURCE_DIR = <rootdir>/variables/src
src:           CMAKE_CURRENT_BINARY_DIR = <rootdir>/variables/build/src
variablesub:   CMAKE_SOURCE_DIR         = <rootdir>/variables
variablesub:   CMAKE_BINARY_DIR         = <rootdir>/variables/build
variablesub:   CMAKE_CURRENT_SOURCE_DIR = <rootdir>/variables/src/variablesub
variablesub:   CMAKE_CURRENT_BINARY_DIR = <rootdir>/variables/build/src/variablesub
src:           CMAKE_CURRENT_SOURCE_DIR = <rootdir>/variables/src
src:           CMAKE_CURRENT_BINARY_DIR = <rootdir>/variables/build/src
top:           CMAKE_CURRENT_SOURCE_DIR = <rootdir>/variables
top:           CMAKE_CURRENT_BINARY_DIR = <rootdir>/variables/build
-- Configuring done (0.4s)
-- Generating done (0.0s)
-- Build files have been written to: <rootdir>/variables/build
```

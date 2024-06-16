# Professional CMake

## Highlights from Part 1 - Getting Started

### The importance of targets
> _"It also establishes the importance of targets, one of the most fundamental concepts in CMake."_ – pg. 4


### Start-to-End Process
```mermaid
flowchart LR
    id1([Configure])-->id2([Generate])-->id3([Build])-->id4([Test])-->id5([Package])
```
CMake helps take care of generating platform-specific project files for the likes of IDEs, testing, and packaging stages respectively, with the overall aim of abstracting away platform differences and making some tasks much simpler.

### Installing CMake fron source
```bash
# Clone repo
$ git clone https://github.com/Kitware/CMake.git && cd $_

# Switch to latest release
$ git switch -c release origin/release

# Build and install
$ ./bootstrap && make && sudo make install

# Verify install was successful
$ cmake --version
cmake version 3.29.3-g3242f4c

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

### Useful links
* [CMake Reference Documentation](https://cmake.org/cmake/help/latest)
* [CMake Forum](https://discourse.cmake.org)

### Setting up a project
> _"Without a build system, a project is just a collection of files."_ – pg. 7

> _"This file [CMakeLists.txt] is a platform-independent description of the whole project, which CMake then turns into platform-specific build tool project files."_ – pg. 7

### Fundamentals
A fundamental part of CMake is the idea of having two directories:
* A "source" directory (location of CMakeLists.txt and source files)
* A "binary" directory (everything produced by the build)

> _"The build tool's project files...are not intended to be put under version control."_ – pg. 7

### Innie or Outie
> _"There are essentially two approaches: in-source (discouraged) and out-of-source builds."_ – pg. 7

The reason in-source builds are discouraged is because the build outputs are intertwined with the source files, making it harder to delete / ignore the outputs and avoid overwriting source files by accident.

It's interesting to see that the auther prefers opting for a parent directory where the source and build directories are children i.e

```bash
$ tree BASE_DIR
BASE_DIR
├── source
│   ├── CMakeLists.txt
│   └── ... source files
└── build
    ├── CMakeLists.txt
    └── ... source files
```
...instead of the build directory existing inside of the source directory.

```bash
$ tree source
source
├── build
│   ├── CMakeLists.txt
│   └── ... source files
├── CMakeLists.txt
└── ... source files
```
The latter is the one I'm more commonly exposed to. It also makes it easy to set up a .gitignore file.

### Generating project files
A variety of file generators are supported (Visual Studio, Xcode, etc., ...) - some even support multiple configurations e.g. "Release", "Debug", and so on.

The most basic way to generate project files is as follows:
```bash
$ mkdir build
$ cd build
$ cmake -G "Unix Makefiles" ../source
```
...or...
```bash
$ mkdir -p build && cd $_
$ cmake .. # if on Linux and not using the parent structure
```

### Configuring and Generating
> _"During the configuring phase, CMake reads the CMakeLists.txt file and builds up an internal representation of the entire project...the generation phase creates the project files."_ – pg. 9

During the generation, a CMakeCache.txt file is created in the build directory, which helps speed up subsequent builds.

`cmake-gui` is also an option instead of running `cmake` on the command line.

### Running the build tool
Previously I would have written something like...
```bash
$ cmake -S . -B debug -DCMAKE_BUILD_TYPE=Debug
$ cmake --build debug
```
..., but it looks like you could supply more flags ad point specifically to the target in "Debug"  for multi-configuration generators.
```bash
cmake --build <path/to/build/folder> --config Debug --target <target-name>
```
If `--target` is not specified, then the default target will be built.

### Recommended Practices
It suggested that developers experiment with different generators, such as Unix Makefiles and Xcode, as projects tend to grow beyond their scope (The Ninja generator was recommended in the book, as it has the broadest platform support of all the generators, and creates very efficient builds).0

### Starting A Simple Project
Below is an example `CMakeLists.txt` file.
```cmake
cmake_minimum_required(VERSION 3.2)
project(MyApp)
add_executable(MyExe main.cpp)
```
Each line in the sample above executes a built-in CMake command.

Arguments can be split by spaces...
```cmake
add_executable(MyExe main.cpp src1.cpp src2.cpp)
```
...or by lines...
```cmake
add_executable(MyExe
    main.cpp
    src1.cpp
    src2.cpp
)
```
Command names are also case-insensitive, but the norm is to write everything lowercase.

### Behave like version....
`cmake_minimum_required()` will always be the first line in any `CMakeLists.txt` file (CMake 3.26 and later will issue a warningif the top level CMakeLists.txt file does not call `cmake_minimum_required()` before calling `project()`).

The typical format of `cmake_minimum_required()` is as follows:
```cmake
cmake_minimum_required(VERSION major.minor[.patch[.tweak]])
```
`VERSION` is mandatory (patch and tweak are not).

At the time of writing this, version 3.5 is the oldest version any user should consider - anythign odler than that will trigger deprecation warnings on the latest verison. It is also said that iOS developers might need to stay very current.

> _"...if the project will itself be a dependency for other projects, then choosing a more recent CMake version may present a hurdle for adoption. In such cases, it may be beneficial to instead require the oldest CMake version that still provides the minimum CMake features needed, ..." – pg. 12

There are techniques for working that out later in the book.

### Dependencies
> _"Dependent projects can always require a more recent version if they so wish, but they cannot require an older one"_ – pg. 13

> _"The main disadvantage of using the oldest workable version is that it may result in more deprecation warnings, since newer CMake versions will warn about older behaviors to encourage maintainers to update their project."_ – pg. 13

### `project()`
```cmake
project(projectName
          [VERSION major[.minor[.patch[.tweak]]]]
          [LANGUAGES languageName ...]
)
```
Every project should have a `project()` function call, and it should appear after `cmake_minimum_required()`.

#### Mandatory
The `projectName` is required, and typically consists of letters, numbers, underscores and hyphens.

#### Optinoal
`VERSION` is only supported from CMake 3.0 and above - it's optional. but a good habit to get into (later chapters go into how we can refer to this version iniformation in the CMakeLists.txt file).

`LANGUAGES` can be separated by spaces e.g.
```cmake
project(myProj C CXX)
```
If no languages are specified, then CMake will default to `C` and `CXX` (C and C++).

New projects are encouraged to use a minimum CMake verison of at least 3.0, and use the new form with the `LANGUAGES` keyword instead.

`project()` also helps to catch compiler and linker issues early - successful checks are then cached to help save time in subsequent runs..

### `add_executable()`
The last step to create our simple project is to add an executable from a selection of source files e.g.
```cmake
# add_executable(<target_name> source1 [source2 ...])
add_executable(myApp main.cpp)
```
Later chapters discuss how to customise the properties of an executable, and multiple executables can also be defined (although using the same target name will result in failure).

### Comments
Comments in CMake are similar to Unix shell scripts - anything after a `#` will result in a comment (unless already in a quoted string).

CMake 3.0 also supports Lua-style block comments with `#[==[` e.g.
```cmake
#[=[
This
is
a
block
comment
]=]
```
You can use any amount of `=` signs (even none), but the opening and closing blocks must match.

### Recommended Practices
If the project is intended to be built and distributed on Linux, it makes sense for the minimum CMake version to be dictated by the version of CMake provided by that same distribution e.g. Ubuntu 20.04 LTS -> CMake Version [3.16](https://launchpad.net/ubuntu/+source/cmake).

### Executable++
The more complete form of `add_executable()` is as follows:
```cmake
add_executable(targetName [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               source1 [source2 ...]
)
```
* `WIN32` - when buildig on a Windows platform, this lets CMake know we are intending to build a Windows GUI application (i.e. with a `WinMain()` entry point, as opposed to the typical `main()` version).
* `MACOSX_BUNDLE` - the directory structure differs when building for macOS, compared to iOS (flatter for iOS); CMake will also generation an `Info.plist` file for bundles.

Both of these options will be ignored if not built on their applicable platforms.

* `EXCLUDE_FROM_ALL` - when no target is specified at build time, the default `ALL` target is build; to exclude a specific target from this group, we can use this option.

### Defining Libraries
Library targets are defined using the `add_library()` function, and has a similar form to `add_executable()`:
```cmake
add_library(targetName [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            source1 [source2 ...]
)
```
`STATIC` - `*.lib` on Windows, `*.a` on Unix-like platforms.
`SHARED` - `*.dll` on Windows, `*dylib` on macOS, and `*.so` on other platforms.
`MODULE` - somewhat like a shared library, but is intended to be loaded at dynamically at runtime, rather than being linked directory to the library or executable.

The preferred practice is to omit this keyword.

> _"In such cases, the library will be either `STATIC` or `SHARED`, with the choice determined by the value of a CMake variable called `BUILD_SHARED_LIBS`. If `BUILD_SHARED_LIBS` has been set to true, the library target will be a shared library, otherwise it will be static"_ – pg. 18

We cam use this variable in combo with the `-D` flag to set it as follows:
```cmake
cmake -DBUILD_SHARED_LIBS=YES /path/to/source
```
We can also set this variable in the CMakeLists.txt file to keep the `cmake` command tidier by using the `set()` command.
```cmake
set(BUILD_SHARED_LIBS YES)
```

### Linking Targets
The author provides good explanations of the various descriptors for linking targets - `PRIVATE`, `PUBLIC`, and `INTERFACE`:

#### `PRIVATE`
Library A uses library B for its own intenral implementation - anything else that links to library A doesn't need to know about library B

#### `PUBLIC`
Library A uses both library B internally AND in its interface i.e. A cannoy be used without B, so anything that uses A will also have a direct dependency on B. The example given in the book is as follows:

> _An example of this would be a function defined in library A which has at least one parameter of a type defined and implemented in library B, so code cannot call the function from A without providing a parameter whose type comes from B""_ – pg. 18

#### `INTERFACE`
In order to use library A, parts of library B must also be used - this differs from `PUBLIC`, because A does not use B internally, only in its interface.

In summary, ask the two following questions:
1) Will A use B internally
2) Will A use B in its interface

The way to link libraries is as follows:
```cmake
target_link_libraries(targetName
    <PRIVATE|PRUBLIC|INTERFACE> item1 [item2 ...]
    [<PRIVATE|PRUBLIC|INTERFACE> item1 [item2 ...]]
    ...
)
```
There are also other `target_...()` commands - these will appear in later chapters, and were introduced from CMake 2.8.11 throguh to 3.2.

Another interesting highlight from this chapter:
> _"..., if using CMake 3.12 or earlier, the `targetName` used with `target_link_libraries()` must have been defined by an `add_executable()` or `add_library()` command in the same directory from which `target_link_libraries()` is being called (this restriction was removed in CMake 3.13)."_ – pg. 19

### Linking non-targets
Apart from linking existing CMake targets, we can also provide...
* Full path to library link (prior to 3.3, you could search for a library by replacing the path, `/usr/lib/libfoo.sh` with `-lfoo`)
* Plain library name (the linker command will search for that library
* Link flag (items starting with a hyphen other than `-l` or `-framework` are treated as flags added to the linker command - these should only really be used

### Older CMake Versions
Previously, you could pass `debug` / `optimized` / `general` to the `target_link_libraries()` command, but this is discouraged now with newer versions.

There was also an older form of `target_link_libraries()`, without the access specificer, but this is also discouraged:
```cmake
target_link_libraries(targetName item [item2 ...]) # almost like `PUBLIC`
```
Another form is...
```cmake
target_link_libraries(targetName
    LINK_INTERFACE_LIBRARIES item [item ...]
)
```
..., which was a pre-cursor to `INTERFACE`, and should be avoided due to its effect on target properties.

And yet one more older version:
```cmake
target_link_libraries(targetName
    <LINK_PRIVATE|LINK_PUBLIC> lib [lib...]
   [<LINK_PRIVATE|LINK_PUBLIC> lib [lib...]]
)
```
Again, opt for the newer `PUBLIC` / `PRIVATE` in its place.

### Recommended Practices
> _"Target names need not be related to the project name."_ – pg. 21

Advice given is to "choose a target name according to what the target does, rather than the project it is part of", as some articles suggest defining a variable for the project name, and then reusing that variable for the name of an executable or library target - this is to be avoided.
```cmake
# BAD: Don't use a variable to set the project name - set it directly
set(projectName MyExample)
project(${projectName})
```
```cmake
# BAD: Don't set the target name from the project name
add_executable(${projectName} ...)
```
```cmake
# GOOD: Set the project name directly
project(MyProj)
```
```cmake
# GOOD: Target name is independent of the project name
add_executable(MyThing ...)
```

Resist the urge to add a `lib` prefix to libraries names, as this will be prepended to the library name by default on most platforms except Windows.

> _"Avoid specifiying the `STATIC` or `SHARED` keyword for a library until it is known to be needed."_ – pg. 22

In this case, opt to take advantage more of `BUILD_SHARED_LIBS` in its place.

Always specify either `PUBLIC`, `PRIVATE`, or `INTERFACE` when calling the `target_link_libraries()` command.

### Testing
`ctest` can be seen as a test scheduling and reporting tool - we would incorporate it in one our of builds as follows:
```cmake
cmake_minimum_required(VERSION 3.19)
project(MyProj VERSION 4.7.2)

enable_testing()

add_executable(testingSomething testSomething.cpp)

add_test(NAME SomethingWorks COMMAND testSomething)
add_test(NAME ExternalTool COMMAND /path/to/tool someArg moreArg)
```

`enable_testing()` is required in order to tell CMake to prepare an input file for `ctest` (typically, it's called after `project()`).

`add_test()` is used to define a test case - the `NAME` + `COMMAND` version is generally recommended.

For `NAME`, CMake 3.19 supports quite a few types of characters, but ideally stick to letters, numbers, hyphens, and underscores.

`COMMAND` can either be a bash command or a target, and - like with most programmes - a return code of `0` signifies success.

The following commands would help run `ctest`:

```cmake
mkdir build
cd build
cmake -G Ninja -DCMAKE_BUILD_TYPE=Debug ../source
cmake --build .
ctest
```
..., which I would probably condense to...
```cmake
cmake -S . -B build
cmake --build build
ctest
```
For multi-config generators, you can be more specific with arguments:
```cmake
cmake -S . -G "Ninja Multi-Config" build
cmake --build build --config Debug
ctest --build-config Debug # or `cmake -C Debug`
```
If working with many tests, we can introduce paralellism:
```cmake
ctest --parallel 16
```
Although, typically, I would write something like:
```cmkae
ctest -j $(nproc)
```
We can get more detailed test feedback by passing the `--verbose` / `-V` flag, or just the output when tests fail with `--output-on-failure`.

> _"Generally, developers are better off learning to run ctest directly and take advantage of all its various options"_ – pg. 24

### Installing

Once we've built files from our project, we can then install them from the build (or source) directory - files can also be tweaked in some shape or form before or after the copy.

We can use the `install()` to take care of this process:
```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProj VERSION 4.7.2)

add_executable(MyApp ...)
add_library(AlgoRuntime SHARED ...)
add_library(AlgoSDK STATIC ...)

# This concise form requires CMake 3.14 or later
install(TARGETS MyApp AlgoRuntime AlgoSDK)
```

> _"When no destinations are given to tell CMake where to install the targets, CMake will use default locations that correspond to the convention used on most Unix systems"_ – pg. 25

The same generally applies for Windows, but Apple's ecosystem is slightly different.

CMake's default layout will install...
* executables -> `bin` subdirectory below the base install location
* libraries in  a `lib` subdirectory
* headers in a `include` subdirectory

You must explcitly specify `DESTINATION` when using CMake 3.13 or earlier:
```cmake
install(TARGETS MyApp AlgoRuntime AlgoSDK
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
```

Files and directories can also be installed:
```cmake
install(FILES things.h algo.h DESTINATION include/myproj)
install(DIRECTORY headers/myproj DESTINATION include)  # recursively copy
install(DIRECTORY headers/mypro/j DESTINATION include) # copy contents of myproj folder with the addition of the `/`
```

In CMake 3.23 and later, we can also use `file sets` to handle headers more easily:
```cmake
add_library(AlgoSDK ...)

target_sources(AlgoSDK
    PUBLIC
        FILE_SET api
        TYPE HEADERS
        BASE_DIR headers
        FILES
            headers/myproj/sdk.h
            headers/myproj/sdk_version.h
)

install(TARGETS AlgoSDK FILE_SET api)
```

CMake 3.14 and later will use `include` as the defualt destination for headers when no destination is provided, and, when using `file sets`, the relative directory structure is retained from the `BASE_DIR` specified:
```bash
$ tree BASE_DIR
BASE_DIR
└── include
    └── myproj
        ├── sdk.h
        └── sdk_version.h
```

> _"When installing libraries and headers for other projects to build against, it is recommended to provide a set of CMake-specific config package files as wel"_ – pg. 27

```cmake
cmake -S . -B debug -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build debug
cmake --install debug --prefix /path/to/somewhere # `--install` available from CMake 3.15
```
If `--prefix` is not given, it will be taken from a platform-dependent value set during the configure step.

You can also use multi-config generators:

```cmake
cmake -S . B build -G "Ninja Multi-Config"
cmake --build build --config Debug
cmake --install build --config Debug --prefix /path/to/somewhere
```

CMake also provides an `install` target, supported on all releases, but it isn't as flexible as `--install`.

As a project grows, a single install package may no longer make sense - at that stage, it would make more sense to look at "Components" (which are addressed in later chapters).

### Packaging
Installing a project from source was so "2023" - it's all about pre-built packages now.

`cpack` takes all the information from the `install()` commands and creates a package specific to your requirements (e.g. `.zip`, `.tar.gz`, `RPM`, `DEB`, etc., ...).

> _"Internally, `cpack` effectively does one or more cmake `--install` commands with `--prefix` set to a temporary staging area.
The contents of that staging area are then used to create a package in the relevant format"_ – pg. 28

Basic packaging is invoked by setting a few variables and including the `CPack` module (this writes out an input file for the `cpak` tool):

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProj VERSION 4.7.2)

add_executable(MyApp ...)
add_library(AlgoRuntime SHARED ...)
add_library(AlgoSDK STATIC ...)

install(TARGETS MyApp AlgoRuntime AlgoSDK)

# These are project-specific
set(CPACK_PACKAGE_NAME MyProj)
set(CPACK_PACKAGE_VENDOR ITHelpDec)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "All your BASE_DIR are belong to us")

# These lines tend to be the same for every project
set(CPACK_PACKAGE_INSTALL_DIRECTORY ${CPACK_PACKAGE_NAME})
set(CPACK_VERBATIM_VARIABLES TRUE)

# This is what writes out the input file for cpack
include(CPack)
```

Another reminder that the `VERSION` keyword in `project()` can be a nice way to embed the version number in source files.

Once we have our `CMakeLists.txt` file in place, we can build and package our project as follow:

```cmake
cmake -S . -B debug -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build debug
cpack --config debug/CPackConfig.cmake -G "ZIP;WIX" # WIX doesn't appear to work on macOS
```

Again, multi-configuration builds also work as follows:
```cmake
cmake -S . -B build -G "Ninja Multi-Config"
cmake --build build --config Release
cpack -G "ZIP;WIX" --config Release # could not test on macOS
```

### `package` target
> _"The project can override the default set by setting the `CPACK_GENERATOR` variable before calling `include(CPack)`"_ – pg. 29

We can use basic flow control to decide how we want to package our project:

```cmake
if(WIN32)
    set(CPACK_GENERATOR ZIP WIX)
elseif(APPLE)
    set(CPACK_GENERATOR TGZ productbuild)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux"))
    set(CPACK_GENERATOR TGZ RPM)
else()
    set(CPACK_GENERATOR TGZ)
endif()
```

I'm surprised there isn't some form of switch statement instead of `if-else` logic.

### Recommended Practices
It's a good idea to do test `install()` commands to a temporary directory using the `--prefix` flag - just make sure the temporary staging area is empty, so it isn't influenced by previous builds.

Consider installing files from the generated packages rather than from the build tree itself - installing directly from a build directory may require administrative privileges, which can cause hard-to-trace build errors.

If the CMake version can be set to at least 3.23, then it is definitely worth investing time into becoming familiar with `file sets`.

* Put all the project's header files in set sets
* Use `PRIVATE` and `PUBLIC` file sets to clearly define which ones are meant to be installed, and which ones are not

> _"This will also simplify the handling of header search paths, both when building the project, and when it is installed."_ – pg. 30

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

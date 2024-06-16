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

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

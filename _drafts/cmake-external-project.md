---
layout: post
category: interest
title: Dependencies in cmake
---

Many C++ projects have external dependencies, but there is no easy way to ensure
that the required dependencies are available at build time. There has been a lot
of work on package managers such as [conan] and [vcpkg], while other projects
use git submodules or just throw an error if dependencies cannot be found.  Many
projects already use the build system generator [cmake], so its built in
functionality can help without requiring developers to add support for a whole
package management system.

The primary way to find and control dependencies in cmake is through
`find_package`, which looks for files matching a dependency and typically
creates targets encapsulating that dependency.  The [ExternalProject] module can
be used alongside `find_package` to automatically download and build any
dependencies that cannot be found locally on the system.

<!--end-excerpt-->

### Find package support

Dependencies in cmake can be found using the [`find_package`][find_package]
command in your CMakeLists.txt files. There are a number of libraries that have
find modules shipped as part of cmake (a list can be found [in the cmake
manual][inbuilt-find]), so using these dependencies is easy. As an example, the
OpenCL library and headers can be found using:

```cmake
find_package(OpenCL REQUIRED)
```

In recent versions of cmake this will provide an imported target
`OpenCL::OpenCL`, as specified in [FindOpenCL]. This target will include the
library to link against and the include directories required to find the OpenCL
headers on the system. Then adding this dependency to an executable target is as
simple as specifying the OpenCL target as a link library:

```cmake
find_package(OpenCL REQUIRED)
add_executable(my_executable ...)
target_link_libraries(my_executable OpenCL::OpenCL)
```

If cmake can find OpenCL in a system directory, then the executable target will
be built with the OpenCL include directory correctly passed to the compiler and
the OpenCL library linked into the executable. However it is possible that a
user has OpenCL in a non-standard location, where cmake will not look by
default. This user can provide the locations of the library and headers to cmake
as additional options:

```bash
cmake -DOpenCL_INCLUDE_DIR=<path/to/headers> \
      -DOpenCL_LIBRARY=<path/to/libOpenCL> \
      <path/to/source>
```

### Writing find modules


### Using ExternalProject

The easiest way to use `ExternalProject` is to call `ExternalProject_Add` with
the URL of the project you want to build in your CMakeLists.txt file.

```cmake
ExternalProject_Add(project_name
    GIT_REPOSITORY https://github.com/path_to/git_repo.git
    GIT_TAG        master
)
```

Then when you run cmake a target called `project_name` will be created which
will download, build and install the project. If the project uses cmake itself,
then the project's cmake files will be used to build the project.
If the external project does not use cmake, then you will have to add the
commands required to build the project yourself in the `ExternalProject_Add`
function, using the `BUILD_COMMAND`, `INSTALL_COMMAND` and so on.

By default this approach will try to install the external dependency so that
the library, headers etc are all available when building the rest of the
project. This can cause problems as this requires the build to have write
permissions to default install locations (e.g. `/usr/local/lib` on linux) and
will affect system state.

It is better not to directly install dependencies but still provide them to
build steps that rely on them. This can be done by using `ExternalProject` to
generate cmake targets that can be added as dependencies to other targets.

### Combining ExternalProject with find modules

### Importing targets

The [googletest] project suggests invoking `cmake --build` within cmake (as
shown in their [documentation][gtest-docs]), so that the `ExternalProject` is
downloaded while cmake is running. This allows the developer to directly use the
googletest cmake configuration using `add_subdirectory(...)`.



[conan]: https://conan.io/
[vcpkg]: https://github.com/microsoft/vcpkg
[cmake]: https://cmake.org
[ExternalProject]: https://cmake.org/cmake/help/latest/module/ExternalProject.html
[find_package]: https://cmake.org/cmake/help/latest/command/find_package.html
[inbuilt-find]: https://cmake.org/cmake/help/latest/manual/cmake-modules.7.htm#find-modules
[FindOpenCL]: https://cmake.org/cmake/help/latest/module/FindOpenCL.html
[googletest]: https://github.com/google/googletest
[gtest-docs]: https://github.com/google/googletest/blob/master/googletest/README.md#incorporating-into-an-existing-cmake-project

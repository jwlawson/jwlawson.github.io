---
layout: post
category: interest
title: Dependencies in cmake
---

Many C++ projects have external dependencies, but there is no easy way to
ensure that the required dependencies are available at build time. There has
been a lot of work on package managers such as [conan] and [vcpkg], while other
projects use git submodules or just throw an error if dependencies cannot be
found. Many projects already use the build system generator [cmake], which
provides built in functionality to help solve this problem without requiring
developers to add support for a package management system.

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
target_link_libraries(my_executable PUBLIC OpenCL::OpenCL)
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

There are many libraries with find modules included with cmake, but what if you
need a dependency that is not? You will need to provide a find module of your
own. The [cmake manual][cmake-find-modules] provides some assistance to help
with this.

When you call `find_package(...)` cmake will look for a find module
corresponding to the requested dependency. The find module has to have a name
matching `Find<lib>.cmake`, where `<lib>` is the library name that will be used
when calling `find_package` (e.g. `find_package(GTest)` looks for
`FindGTest.cmake`). These find modules need to be available in the
`CMAKE_MODULE_PATH`, so you might need to add the directory containing the
modules to this list in your CMakeLists.txt.

```cmake
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules/")
```

A typical find module will contain a call to [`find_path`][find_path] which
looks for the libraries header files, and a call to
[`find_library`][find_library] which looks for the library itself.  Keeping with
the example of OpenCL, if we were to implement our own find module it would need
to include the following:

```cmake
find_library(OpenCL_LIBRARY
  NAMES OpenCL
)
find_path(OpenCL_INCLUDE_DIR
  NAMES CL/cl.h
)
```

The `OpenCL_LIBRARY` and `OpenCL_INCLUDE_DIR` variables are the same as those
used above for a user to specify the location of the OpenCL library.  If these
variables are not defined, then cmake will look for the OpenCL library and
headers. If these variables are already defined when `find_*` is called then
cmake assumes that the path in the variable is correct and will not search
further.

However if cmake fails to find the library or headers, then the variables are
left undefined. Most find modules make use of the
`FindPackageHandleStandardArgs` module to consistently handle cases like this.
To do this, the find module should include something like:

```cmake
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(OpenCL
  FOUND_VAR OpenCL_FOUND
  REQUIRED_VARS OpenCL_LIBRARY OpenCL_INCLUDE_DIR
)
```

This tells cmake which variables have to be defined in order for the dependency
to be met and sets `OpenCL_FOUND` to `true` if it is available or `false` if
not. If a user calls `find_package(OpenCL REQUIRED)` and cmake cannot find the
library or headers then the `find_package_handle_standard_args` function call
will output an error.

Now that we know whether or not the dependency has been found and we know the
paths to the headers and the library this information needs to made available to
users. This is best done by creating a new target that incorporates these
imported paths:

```cmake
if(OpenCL_FOUND AND NOT TARGET OpenCL::OpenCL)
  add_library(OpenCL::OpenCL UNKNOWN IMPORTED)
  set_target_properties(OpenCL::OpenCL PROPERTIES
    IMPORTED_LOCATION "${OpenCL_LIBRARY}"
    INTERFACE_INCLUDE_DIRECTORIES "${OpenCL_INCLUDE_DIR}"
  )
endif()
```

This imported target can be treated in just the same way as the `OpenCL::OpenCL`
target provided by cmake's built in FindOpenCL module. In fact you can look at
the [FindOpenCL source code] to see that under the hood this is all that cmake
is doing (albeit with more platform specific workarounds).

Another full example is included in [SYCL-DNN], which uses Google benchmark as
its benchmarking framework. Its [find module][findbenchmark] is very straight
forward and closely follows the above.


### Using ExternalProject

Find modules are really good for finding dependencies already installed on a
system, but often a project has external dependencies that are unlikely to be
installed and a user may not actually want to or be able to install them. In
cases like this we can instruct cmake to download its own copy of the
dependencies and use that instead.

The easiest way to do this is to use the [ExternalProject] cmake module. This
contains the `ExternalProject_Add` function which will create a new target to
fetch and build an external dependency:

```cmake
ExternalProject_Add(project_name
    GIT_REPOSITORY https://github.com/path_to/git_repo.git
    GIT_TAG        master
)
```

When you run cmake a target called `project_name` will be created which
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
build steps that rely on them. To do this you can specify `INSTALL_COMMAND ""`
to prevent cmake trying to install the library, and then create an imported
target using the built library and downloaded headers.

Continuing with the OpenCL example, the Khronos group provides an open source
OpenCL ICD loader library, as well as OpenCL headers. These can be used together
to provide the OpenCL dependency:

```cmake
include(ExternalProject)
ExternalProject_Add(opencl_headers
  GIT_REPOSITORY    https://github.com/KhronosGroup/OpenCL-Headers
  GIT_TAG           master
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ""
)
ExternalProject_Get_Property(opencl_headers SOURCE_DIR)
file(MAKE_DIRECTORY ${SOURCE_DIR})
set(OpenCL_INCLUDE_DIR ${SOURCE_DIR}
  CACHE PATH "OpenCL header directory" FORCE
)

ExternalProject_Add(opencl_icd_loader
  GIT_REPOSITORY  https://github.com/KhronosGroup/OpenCL-ICD-Loader
  GIT_TAG         master
  DEPENDS         opencl_headers
  CMAKE_ARGS      -DOPENCL_ICD_LOADER_HEADERS_DIR=${OpenCL_INCLUDE_DIR}
  INSTALL_COMMAND ""
)
ExternalProject_Get_Property(opencl_icd_loader BINARY_DIR)
set(OpenCL_LIBRARY ${BINARY_DIR}/libOpenCL.so
  CACHE PATH "OpenCL library location" FORCE
)
```

External project will download and build the headers and library, but we cannot
link them to other cmake targets as we have not created a cmake target that
contains the OpenCL dependency. We could explicitly add calls to `add_library`
etc here, but note that this would be identical to the target creation in the
find module. Rather than repeating ourselves we can simply set the `OpenCL_*`
cache variables to point to the build artifacts and call `find_package(OpenCL)`
to generate the `OpenCL::OpenCL` target as before.

This new target will not know that it relies on the external projects being
built, so one last thing to do is add dependencies to ensure that everything is
built in the correct order:

```cmake
add_dependencies(OpenCL::OpenCL opencl_headers opencl_icd_loader)
```

### Playing nice with package managers

Even if you don't want to provide support for a package manager, your users
might. Equally they might like to use your project as a dependency
within their own project using a package manager. As a result it is good
practise to keep this use-case in mind when developing your build
scripts.

Package managers will want to manage all dependencies for projects they are
trying to build, which can clash with the above use of ExternalProject, causing
problems such as the same dependency being provided and built multiple times,
version clashes and linking errors.  To avoid these problems and coexist with
package managers all calls to ExternalProject as above should be wrapped in an
option that allows a user to disable downloading and building any external
dependencies.

```cmake
option(DOWNLOAD_DEPENDENCIES
  "Whether to download any dependencies not found on the system"
  ON
)
```

This allows users of package managers to disable this functionality as the
dependencies will all be provided by their package manager.


### Putting it all together

Combining ExternalProject and find modules provides a powerful way to ensure
that external dependencies can be found for cmake projects. But a user might not
want to always download and build libraries that are available on their systems.
It might be better to try and find the library locally, falling back to the
option of downloading it if not found. This can easily be done with a simple
cmake script:

```cmake
find_package(OpenCL QUIET)

if(NOT OpenCL_FOUND AND DOWNLOAD_DEPENDENCIES)
  # Use ExternalProject as above
  include(ExternalProject)
  ExternalProject_Add(opencl_headers
    ...
  )
  ...
  # Create library target using newly set up dependency
  find_package(OpenCL REQUIRED)
  add_dependencies(OpenCL::OpenCL opencl_headers opencl_icd_loader)
endif()
```

If OpenCL is found on the system, then the first call to `find_package` will
find it and will set `OpenCL_FOUND` to `TRUE`. If it is not available on the
system then we go through the ExternalProject setup as above to download and
build OpenCL before calling `find_package` again with the paths set to the build
directory.

The [clinfo-lite] projects gives a full example of how this all works.


[conan]: https://conan.io/
[vcpkg]: https://github.com/microsoft/vcpkg
[cmake]: https://cmake.org
[ExternalProject]: https://cmake.org/cmake/help/latest/module/ExternalProject.html
[find_package]: https://cmake.org/cmake/help/latest/command/find_package.html
[inbuilt-find]: https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#find-modules
[FindOpenCL]: https://cmake.org/cmake/help/latest/module/FindOpenCL.html
[clinfo-lite]: https://github.com/jwlawson/clinfo-lite
[cmake-find-modules]: https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html#find-modules
[find_path]: https://cmake.org/cmake/help/latest/command/find_path.html
[find_library]: https://cmake.org/cmake/help/latest/command/find_library.html
[FindOpenCL source code]: https://github.com/Kitware/CMake/blob/master/Modules/FindOpenCL.cmake
[SYCL-DNN]: https://github.com/codeplaysoftware/SYCL-DNN
[findbenchmark]: https://github.com/codeplaysoftware/SYCL-DNN/blob/master/cmake/Modules/Findbenchmark.cmake

[googletest]: https://github.com/google/googletest
[gtest-docs]: https://github.com/google/googletest/blob/master/googletest/README.md#incorporating-into-an-existing-cmake-project

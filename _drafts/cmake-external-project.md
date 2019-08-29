---
layout: post
category: interest
title: Two approaches to cmake's ExternalProject
---

The build system generator `cmake` has an incredibly useful module to download,
build and install third party projects called [ExternalProject].

One of the major problems in modern programming is that almost all projects rely
on a number of third party libraries. Some programming languages have fancy
package managers to handle these dependencies, while others rely on the project
maintainers to provide some way of handling them. This could be done through the
use of git submodules or just throwing errors if the dependencies can't be found
on a system. Another approach is to use `cmake` to automatically download and
build any missing dependencies.

<!--end-excerpt-->

### The basic idea

The easiest way to use `ExternalProject` is to call `ExternalProject_Add` with
the URL of the project you want to build in your CMakeLists.txt file.

```
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

### Importing targets

The [googletest] project suggests invoking `cmake --build` within cmake (as
shown in their [documentation][gtest-docs]), so that the `ExternalProject` is
downloaded while cmake is running. This allows the developer to directly use the
googletest cmake configuration using `add_subdirectory(...)`.



[ExternalProject]: https://cmake.org/cmake/help/latest/module/ExternalProject.html
[googletest]: https://github.com/google/googletest
[gtest-docs]: https://github.com/google/googletest/blob/master/googletest/README.md#incorporating-into-an-existing-cmake-project

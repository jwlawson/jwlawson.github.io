---
layout: post
category: interest
title: Two approaches to cmake's ExternalProject
---

The build system generator `cmake` has an incredibly useful module to download,
build and install third party projects called
[ExternalProject][cmake-ExternalProject].

One of the major problems in modern programming is that almost all projects rely
on a number of third party libraries. Some programming languages have fancy
pacakge managers to handle these dependencies, while others rely on the project
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
    URL https://some_url.com/to_the/project_sources.tar.gz
)
```
Or if the project uses git, you can specify the git repository and tag:
```
ExternalProject_Add(project_name
    GIT_REPOSITORY https://github.com/path_to/git_repo.git
    GIT_TAG        master
)
```

Then when you run cmake a target called `project_name` will be created which
will download, build and install the project. If the project uses cmake itself,
then this is all you ned to do, as the project's cmake files will be used to
build the project.

If the external project does not use cmake, then you will have to add the
commands required to build the project yourself in the `ExternalProject_Add`
function, using the `BUILD_COMMAND`, `INSTALL_COMMAND`, etc parameters.

### The two approaches

There are two main approaches you can take when it comes to using
ExternalProject. By default 

[cmake-ExternalProject]: https://cmake.org/cmake/help/latest/module/ExternalProject.html

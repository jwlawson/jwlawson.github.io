---
layout: main
title: Quiver suite
---

Page under construction.

Suite of programs to perform calculations with quivers.

### [libqv]

`libqv` is the main library which contains the algorithms and data structures
which the other programs all depend upon.

This contains a suite of unit tests which run using [googletest]. The googletest
libraries must be installed should you wish to run these tests. Otherwise the
library can be built without building the tests.

### [qvtrim]

`qvtrim` is a program to filter the output of other qv* programs to extract
useful information. Full details can be found on the project page.

The main use of this is to filter an output to prevent any repeated matrices
from being shown, or to extract all matrices which are mutation-equivalent.

### [qvmmi]

`qvmmi` is a program designed to run using MPI. This allows it to run on large
distributed systems such as supercomputers.

The program takes a file of mutation-finite quivers as input, tries to add a
vertex to each in as many ways as it can and checks if the resulting quivers are
minimal mutation-infinite quivers.

The parallelization is achieved by having a single master node read the input
and add the vertices to each quiver. The master then passes the resulting
quivers to the slave nodes to check whether they are minimal mutation-infinite.

### [qvfin]

`qvfin` constructs all mutation-finite quivers of a given size. The program
itself is a script which relies upon `qvtrim` and a program `qvfex` included in
qvfin project page.  

`qvfex` checks all possible mutation-finite extensions of each matrix provided
to it and outputs any which are mutation-finite.

### [qvmove]

### [qvdraw]

`qvdraw` is a script which uses [gml2pic] and the [OGDF] library to draw
pictures of the quivers output from qv* programs.

The script uses two small programs `qv2gml` and `gmlayout` to construct a file
to pass to `gml2pic`. These are available on the `qvdraw` project page.


[libqv]: https://github.com/jwlawson/qv
[qvtrim]: https:://github.com/jwlawson/qvtrim
[qvmmi]: https://github.com/jwlawson/qvmmi
[qvfin]: https://github.com/jwlawson/qvfin
[qvmove]: https://github.com/jwlawson/qvmove
[qvdraw]: https://github.com/jwlawson/qvdraw
[ogdf]: http://www.ogdf.net/ogdf.php
[gml2pic]: http://www.ogdf.net/doku.php/project:gml2pic
[googletest]: https://code.google.com/p/googletest/

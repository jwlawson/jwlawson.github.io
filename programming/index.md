---
layout: main
title: Programming
---

### Quiver computations

Throughout my PhD I developed a suite of tools to compute mutations of
quivers. More information on the uses of these tools can be found on the
[maths page], with more complete information on the [quiver software page].

[libqv]
: The main library containing most of the computations, data structures and graph
objects.

[qvfin]
: Tool to generate all mutation-finite quivers of a given size.
: All subquivers of a mutation-finite quiver must also be mutation-finite, so
candidates for mutation-finite quivers of a given size can be constructed by
adding a vertex to all smaller mutation-finite quivers. The size of each
candidate's mutation class can then be computed to determine whether it is
finite or not. As a single vertex quiver's mutation class only contains itself
we can recursively compute all mutation-finite quivers of any size.

[qvmmi]
: MPI based parallel executable to compute every minimally mutation-infinite
quiver.
: A minimally mutation-infinite quiver is a quiver which is mutation-infinite,
but every subquiver is mutation-finite. There are many possible candidates so a
parallel method to check many quivers simultaneously scaled well across many
cores.

[qvtrim]
: A CLI tool to filter streams of quivers.
: Tools such as [qvmmi] and [qvfin] generate large text streams containing
representations of quivers. Due to the design and parallel nature of these
programs these streams will typically contain duplicates and quivers which are
identical up to permutation of their vertices. The [qvtrim] tools allows a user
to filter these streams to only show quivers that are useful to them.

[qvdraw]
: Tools to draw quivers, exchange graphs and mutation classes
: As quivers can be represented as directed graphs, and their exchange graphs
have interesting geometric properties is it useful to be able to visualise them.
The tools in [qvdraw] can layout the graph and output as a picture or as LaTeX to
allow images to be embedded in papers.

### N-of-1 trial app

In the summer of 2012 I developed a proof of concept Android app to aid
clinicians running N-of-1 trials.  The project has now ended and the software is
no longer maintained, but the source is still available on [Github][nof1
github].

### Ludum Dare

I have participated in a number of the game creation competitions Ludum Dare.
Write ups for these are on the [Ludum Dare page][ludum dare].

[maths page]: {{ site.baseurl }}/maths

[quiver software page]: {{ site.baseurl }}/maths/mmi/software
[libqv]: https://github.com/jwlawson/qv
[qvmmi]: https://github.com/jwlawson/qvmmi
[qvtrim]: https://github.com/jwlawson/qvtrim
[qvfin]: https://github.com/jwlawson/qvfin
[qvdraw]: https://github.com/jwlawson/qvdraw

[nof1 github]: //github.com/jwlawson/nof1

[ludum dare]: {{ site.baseurl }}/ludumdare

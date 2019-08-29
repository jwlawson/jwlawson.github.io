---
layout: main
title: John Lawson
---
{% include base.html %}

Working on accelerating machine learning on a range of hardware using the [SYCL]
parallel programming model. Designed and led development of [SYCL-DNN], worked on
the SYCL backend for TensorFlow and Eigen, now leading a squad to build new
features across the [SYCL Ecosystem].

### PhD Research

Completed my PhD at Durham in 2017, looking at combinatorial and geometric
properties of cluster algebras, supervised by [Pavel Tumarkin].  More
information is available [on my archived research pages][maths page]. The full
PhD thesis is available on [Durham e-Theses][thesis].

### Side projects and interesting things

The [programming page] gives an overview of some of the things I have worked
on, and many of these projects are available on [GitHub].

<ul id="side-projects" class="list-unstyled">
  {% for post in site.categories.interest %}
  {% unless post.sub %}
    <li class="panel panel-info">
      <div class="panel-heading"><a href="{{ base }}{{ post.url }}">{{ post.title }}</a></div>
      <div class="panel-body">{{ post.excerpt | markdownify }}</div>
    </li>
  {% endunless %}
  {% endfor %}
</ul>

[SYCL]: https://www.khronos.org/sycl/
[SYCL-DNN]: https://github.com/codeplaysoftware/SYCL-DNN
[SYCL-BLAS]: https://github.com/codeplaysoftware/SYCL-BLAS
[SYCL Ecosystem]: https://developer.codeplay.com/home/

[maths page]: {{ base }}/maths
[thesis]: http://etheses.dur.ac.uk/12095/
[Pavel Tumarkin]: http://www.maths.dur.ac.uk/users/pavel.tumarkin/

[GitHub]: https://github.com/jwlawson
[programming page]: {{ base }}/programming

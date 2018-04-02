---
layout: main
title: John Lawson
---

Mathematician pretending to be a software engineer.

Working on accelerating machine learning on a range of hardware using the SYCL
application model, primarily through actively contributing to the SYCL backend
of Tensorflow and Eigen.

<div class="alert alert-info" role="alert">
<strong>Heads up:</strong> This site hasn't been updated much recently, so may
contain information which is a little dated and inaccurate. I have finished my
Phd, am no longer working in academia or teaching, and should hopefully have
this updated sometime.
</div>

{% include base.html %}
<div class="row triple-col">
<div class="col-sm-4">
  <a class="thumbnail" href="{{ base }}/maths">
  <i class="fa fa-flask fa-4x"></i>
  <h3>Research</h3>
  <p>Discover recent research.</p>
  </a>
</div>
<div class="col-sm-4">
  <a class="thumbnail" href="{{ base }}/teaching">
  <i class="fa fa-university fa-4x"></i>
  <h3>Teaching</h3>
  <p>Resources related to tutorials.</p>
  </a>
</div>
<div class="col-sm-4">
  <a class="thumbnail" href="{{ base }}/programming">
  <i class="fa fa-code fa-4x"></i>
  <h3>Programs</h3>
  <p>Recent programming projects.</p>
  </a>
</div>
</div>

### Side projects and interesting things
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

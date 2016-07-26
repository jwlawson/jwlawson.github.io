---
layout: main
title: John Lawson
---

This is the personal website of John Lawson.

I am a PhD student working at Durham University, supervised by Pavel Tumarkin 
and John Parker. The main focus of my research and work is in cluster algebras 
and the geometric applications of them.

{% include base.html %}
<div class="row triple-col">
<div class="col-sm-4">
  <a class="thumbnail" href="http://www.maths.dur.ac.uk/users/j.w.lawson/">
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
			<div class="panel-heading"><a href="{{ post.url }}">{{ post.title }}</a></div>
			<div class="panel-body">{{ post.excerpt | markdownify }}</div>
		</li>
	{% endunless %}
	{% endfor %}
</ul>

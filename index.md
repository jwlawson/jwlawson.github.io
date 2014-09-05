---
layout: main
title: John Lawson
---

<ul id="postlist">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> &#8212; <span>{{ post.date | date: "%-d %B %Y" }}</span>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>

<div class="triple-col"><!--
 --><div class="column-left">
    <a href="{{ site.baseurl }}/research">
    <i class="fa fa-flask fa-4x fa-border"></i>
    <h3>Research</h3>
    <p>Discover recent research.</p>
    </a>
  </div><!--
 --><div class="column-center">
    <a href="{{ site.baseurl }}/teaching">
    <i class="fa fa-university fa-4x fa-border"></i>
    <h3>Teaching</h3>
    <p>Resources related to tutorials.</p>
    </a>
  </div><!--
 --><div class="column-right">
    <a href="{{ site.baseurl }}/programming">
    <i class="fa fa-code fa-4x fa-border"></i>
    <h3>Programming</h3>
    <p>Links to recent programming projects.</p>
    </a>
  </div><!--
 --></div>

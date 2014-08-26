---
layout: main
title: Ludum Dare
---

Ludum Dare is a game creation competition which runs every 4 months and 
challeges designers, programmers and artists to create a working game in just 48 
hours. See their [webpage](http://www.ludumdare.com/compo/rules/) for more 
information.

Over the last few years I have created games for the competition when I could
fit it around other commitments.

<ul id="postlist">
  {% for post in site.categories.ludumdare %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> &#8212; <span>{{ post.date | date: "%-d %B %Y" }}</span>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>

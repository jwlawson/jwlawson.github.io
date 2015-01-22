---
layout: main
title: Travel
---

<ul id="travellist">
  {% for post in site.categories.travel %}
    <li>
      <a href="{{ post.conf_url }}">{{ post.title }}</a> &#8212;
			<span>{{ post.conf_dates }}</span>
      {{ post.content }}
    </li>
		<hr></hr>
  {% endfor %}
</ul>

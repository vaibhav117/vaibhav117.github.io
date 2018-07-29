---
title: "Welcome to my blog"
layout: defaults
---

<ul>
  {% for post in site.posts %}
    <li>
      <p class="post-meta">{{ post.date | date: site.date_format }}</p>
      <a href="{{ post.url }}"><h2> {{ post.title }} </h2></a>
    </li>
  {% endfor %}
</ul>
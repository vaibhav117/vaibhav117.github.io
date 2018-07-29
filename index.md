---
title: "Welcome to my blog"
layout: defaults
---

<ul>
  {% for post in site.posts %}
    <li>
      <p class="post-meta">{{ post.date | date: site.date_format }}
        <a href="{{ post.url }}"><h3 style:"color:blue">{{ post.title }} </h3></a>
      </p>
    </li>
  {% endfor %}
</ul>
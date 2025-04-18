---
layout: default
title: Home
---

<h1>Welcome to my blog</h1>

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ site.baseurl  }}{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%b %d, %Y" }}
    </li>
  {% endfor %}
</ul>
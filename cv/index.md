---
layout: default
title: CV
permalink: /cv/
---

<ul>
  {% for post in site.cv %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> — {{ post.date | date: "%b %-d, %Y" }}
    </li>
  {% endfor %}
</ul>
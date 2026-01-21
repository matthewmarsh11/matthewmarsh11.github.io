---
title: "Blog"
permalink: /blog/
author_profile: false
classes: wide
---

Welcome to my blog! Here you might find some interesting technical projects I have done outside of my PhD; as well as some reviews on food I have either cooked or eaten in various restaurants around London or Liverpool, and other random thoughts, probably about football and other sport in general.

---

## Technical / Side Projects
{% assign tech_posts = site.posts | where_exp: "post", "post.categories contains 'technical'" %}
{% if tech_posts.size > 0 %}
{% for post in tech_posts %}
**[{{ post.title }}]({{ post.url }})** - *{{ post.date | date: "%b %Y" }}*
{{ post.excerpt | strip_html | truncate: 120 }}

{% endfor %}
{% else %}
*Coming soon...*
{% endif %}

---

## Food Reviews
{% assign food_posts = site.posts | where_exp: "post", "post.categories contains 'food'" %}
{% if food_posts.size > 0 %}
{% for post in food_posts %}
**[{{ post.title }}]({{ post.url }})** - *{{ post.date | date: "%b %Y" }}*
{{ post.excerpt | strip_html | truncate: 120 }}

{% endfor %}
{% else %}
*Coming soon...*
{% endif %}

---

## General Rambling
{% assign general_posts = site.posts | where_exp: "post", "post.categories contains 'general'" %}
{% if general_posts.size > 0 %}
{% for post in general_posts %}
**[{{ post.title }}]({{ post.url }})** - *{{ post.date | date: "%b %Y" }}*
{{ post.excerpt | strip_html | truncate: 120 }}

{% endfor %}
{% else %}
*Coming soon...*
{% endif %}

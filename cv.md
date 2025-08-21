---
layout: page
title: "CV"
permalink: /cv/
---

You can view my CV below, or [download the PDF here](/assets/matthew-marsh-cv.pdf).

{% for item in site.cv %}
  {{ item.content }}
{% endfor %}
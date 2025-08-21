---
layout: page
title: "CV"
permalink: /_cv/
---

You can view my CV below, or [download the PDF here](/assets/matthew-marsh-cv.pdf).

{% for item in site._cv %}
  {{ item.content }}
{% endfor %}
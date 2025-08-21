---
layout: default
title: CV
permalink: /cv/
---

<p><a href="{{ '/assets/cv.pdf' | relative_url }}" download>Download CV (PDF)</a></p>

<object data="{{ '/assets/cv.pdf' | relative_url }}" type="application/pdf" width="100%" height="900px">
  <p>Your browser can't display PDFs. Please download it
     <a href="{{ '/assets/cv.pdf' | relative_url }}">here</a>.
  </p>
</object>

<hr>

## Markdown CV

{% assign md_cv = site.cv | first %}
{% if md_cv %}
{{ md_cv.content }}
{% else %}
<p>Markdown CV not found. Add one at <code>_cv/cv.md</code>.</p>
{% endif %}
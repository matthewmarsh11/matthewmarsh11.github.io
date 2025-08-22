---
layout: default
title: CV
permalink: /cv/
---

# CV

{% assign md_cv = site.cv | first %}
{% if md_cv %}
{{ md_cv.content }}
{% else %}
<p>Markdown CV not found. Add one at <code>_cv/cv.md</code>.</p>
{% endif %}

<p>
  <a href="{{ '/assets/matthew-marsh-cv.pdf' | relative_url }}" download class="download-button">
    Here's a full PDF
  </a>
</p>

<style>
.download-button {
  display: inline-block;
  padding: 0.75rem 1.5rem;
  background-color: #007acc;
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-weight: bold;
}
.download-button:hover {
  background-color: #005fa3;
}
</style>
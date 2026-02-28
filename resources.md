---
layout: default
title: Resources
---

## Resources

<ul>
  {% for resource in site.resources %}
    <li>
      <a href="{{ resource.url | relative_url }}">{{ resource.title }}</a>
    </li>
  {% endfor %}
</ul>

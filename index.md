---
layout: default
title: Montreal Grammars and Parsing Project
---

<div class="main">
  <h1>Montreal Grammars and Parsing Project</h1>
  <span class="authors">Timothy J. O'Donnell, Chris Bruno, Daniel Harasim, Emily Kellison-Linn Eva Portelance and Elias Stengel-Eskin</span>
</div>

### Chapters
{% assign sorted_pages = site.pages | sort:"name" %}
{% for p in sorted_pages %}
    {% if p.layout == 'chapter' %}
- [{{ p.title }}]({{ site.baseurl }}{{ p.url }})<br>
    <em>{{ p.description }}</em>
    {% endif %}
{% endfor %}

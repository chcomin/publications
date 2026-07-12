---
layout: default
title: Publications
---
<h1>{{ site.title }}</h1>

<p>{{ site.description }}</p>

<ul>
  {% assign articles = site.articles | sort: "date" | reverse %}
  {% for article in articles %}
  <li>
    <a href="{{ article.url | relative_url }}">{{ article.title }}</a>
    —
    {% for author in article.authors %}{{ author.name }}{% unless forloop.last %}, {% endunless %}{% endfor %}
    — <em>{{ article.journal }}</em>, {{ article.year }}
  </li>
  {% endfor %}
</ul>

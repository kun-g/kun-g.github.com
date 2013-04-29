---
layout: homepage
title: 为道
---
<h2>{{ page.title }}</h2>
<p> 最新文章 </p>
<ul>
  {% for p in site.posts %}
    <li>{{ p.date | date_to_string }} <a href="{{ p.url }}"> {{ p.title }}</a></li>
  {% endfor %}
</ul>

<p> Categories </p>
<ul>
  {% for p in site.categories.CATEGORY %}
    <li>{{ p.date | date_to_string }} <a href="{{ p.url }}"> {{ p.title }}</a></li>
  {% endfor %}
</ul>

<p> Tags </p>
<ul>
  {% for p in site.tags.TAG %}
    <li>{{ p.date | date_to_string }} <a href="{{ p.url }}"> {{ p.title }}</a></li>
  {% endfor %}
</ul>
<p>Perpage {{ paginator.per_page }}</p>
<p>posts {{ paginator.total_posts }}</p>
{{site.time}}


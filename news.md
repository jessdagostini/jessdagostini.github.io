---
layout: default
title: News
permalink: /news/
---

<h2 style="margin: 60px 0px 10px;">News</h2>

<ul class="news">
  {% for news in site.data.news.main %}
  <li>
    <em>[{{ news.date }}]</em> - <strong>{{ news.title }}</strong><br />
    {{news.content}}
  </li>
  {% endfor %}
</ul>

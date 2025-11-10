---
layout: default
title: Home
---

<h1 id="about-me"></h1>

<h2 style="margin: 80px 0px 10px;"></h2>

I'm a 4th year Ph.D. Student at the University of California Santa Cruz and [2024 ACM SIGHPC Fellow](https://www.sighpc.org/opportunities/fellowships/2024-fellowship-winners), advised by Prof. Dr. [Abel Souza](https://asouza.io). My research interests focus on advancing computational methods for scientific applications by integrating workload characterization, predictive modeling, load balancing optimizations, and intent-based resource management within high-performance computing environments. During my PhD I have characterized a Pangenomics mapping application and developed [miniGiraffe](https://github.com/jessdagostini/miniGiraffe), a proxy application for pangenomics mapping that allows computer scientists to explore different optimizations for this new type of workload. I am also enthusiastic about graph-related problems. I love programming and solving logical puzzles (and trying to do lettering art)!

<h2 style="margin: 60px 0px 10px;">Awards</h2>

<ul class="awards">
  {% for award in site.data.awards.main %}
  <li>
    [{{ award.date }}] <strong>{{ award.title }}</strong>
  </li>
  {% endfor %}
</ul>

<h2 style="margin: 60px 0px 10px;">News</h2>

<ul class="news">
  {% for news in site.data.news.main limit: site.front_page_news %}
  <li>
    <em>[{{ news.date }}]</em> - <strong>{{ news.title }}</strong><br />
    {{news.content}}
  </li>
  {% endfor %}
</ul>

{% assign numposts = site.data.news.main | size %}
{% if numposts >= 1 %}
[Older News]({{ site.base }}/news)
{% endif %}

<!-- {% include_relative _includes/contact.md %} -->

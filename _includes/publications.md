<h1 id="publications"></h1>

<h2 style="margin: 60px 0px -15px; font-size: 40px;">Publications </h2>


<div class="publications">
<h3 style="font-size: 20px;">Conferences</h3>
<ol class="bibliography">
{% for link in site.data.publications.conferences %}
<li>
<div class="pub-row">
  <div class="col-sm-3">
      <abbr class="badge">{{ link.conference_short }}</abbr>
  </div>
  <div class="col-sm-9">
      <div class="title"><a href="{{ site.baseurl }}/assets/papers/{{ link.pdf }}">{{ link.title }}</a></div>
      <div class="author">{{ link.authors }}</div>
      <div class="periodical"><em>{{ link.conference }}</em>
      </div>
    <div class="links">
      {% if link.pdf %}
      <a href="{{ site.baseurl }}/assets/papers/{{ link.pdf }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">PDF</a>
      {% endif %}
      {% if link.doi %} 
      <a href="{{ link.doi }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">DOI</a>
      {% endif %}
      {% if link.code %} 
      <a href="{{ link.code }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Code</a>
      {% endif %}
      {% if link.page %} 
      <a href="{{ link.page }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Page</a>
      {% endif %}
      {% if link.artifact %} 
      <a href="{{ link.artifact }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Artifact</a>
      {% endif %}
      {% if link.notes %} 
      <strong> <i style="color:#e74d3c; font-weight:600">{{ link.notes }}</i></strong>
      {% endif %}
      {% if link.others %} 
      {{ link.others }}
      {% endif %}
    </div>
  </div>
</div>
</li>

{% endfor %}

</ol>
</div>



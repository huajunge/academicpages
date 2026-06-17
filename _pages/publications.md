---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

{% if author.googlescholar %}
  You can also find my articles on <u><a href="{{author.googlescholar}}">my Google Scholar profile</a>.</u>
{% endif %}

{% include base_path %}

{% assign publications = site.publications | sort: "date" | reverse %}
{% assign current_year = "" %}
{% assign publication_index = 0 %}

<div class="publication-list">
{% for post in publications %}
  {% assign publication_year = post.date | default: "1900-01-01" | date: "%Y" %}
  {% if publication_year != current_year %}
    {% unless forloop.first %}
      </ol>
    </section>
    {% endunless %}
    <section class="publication-year-group">
      <h2 class="publication-year-heading">{{ publication_year }}</h2>
      <ol class="publication-year-list" start="{{ publication_index | plus: 1 }}">
    {% assign current_year = publication_year %}
  {% endif %}

  {% assign publication_index = publication_index | plus: 1 %}
  {% include publication-list-item.html %}

  {% if forloop.last %}
      </ol>
    </section>
  {% endif %}
{% endfor %}
</div>

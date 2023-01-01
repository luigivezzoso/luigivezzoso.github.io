---
layout: page
permalink: /archive
title: Archive of All Collections
---

**Note:** This page contains all material in anything in `_posts`, as well as all custom collections, such as `_notes`, and `_journals`, etc. with the only exception being `_pages`.

<!-- 
{% for collection in site.collections %}
{% if collection.label != "pages" %}

  <h2>Entries from {{ collection.label | capitalize }}</h2>
  <ul>
    {% for item in site[collection.label] %}
      <li class="archive-links"><a href="{{ item.url }}">{{ item.title }}</a></li>
    {% endfor %}
  </ul>
  {% endif %}
{% endfor %}

-->

{% for post in site.posts %}

        {% capture year_of_current_post %}
        {{ post.date | date: "%Y" }}
        {% endcapture %}

        {% capture year_of_previous_post %}
        {{ post.previous.date | date: "%Y" }}
        {% endcapture %}

        {% if forloop.first %}
        <h2>{{ year_of_current_post }}</h2>
        <ul>
        {% endif %}

                <li><a href="{{ post.url }}">{{ post.title }}</a></li>

        {% if forloop.last %}
        </ul>
        {% else %}
        {% if year_of_current_post != year_of_previous_post %}
        </ul>

        <h2>{{ year_of_previous_post }}</h2>
        <ul>
        {% endif %}
        {% endif %}

{% endfor %}
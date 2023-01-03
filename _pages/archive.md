---
layout: page
permalink: /archive
title: Posts Archive
---

{% for post in site.posts %}

  {% capture current_year %}

    {{post.date | date: "%Y"}}

  {% endcapture %} 

  {% if current_year != previous_year %} 

    {% assign previous_year = current_year %}

  <h3 class="post-header">
      <span role="img" aria-label="icon-book" aria-hidden="true"></span>
     {{ current_year }}
   </h3>
  {% endif %}
  <!-- <article class="posts">
    <span class="posts-date">{{ post.date | date: "%b %d" }}</span><header class="posts-header"> <h4 class="posts-title"> <a href="{{ post.url }}">{{ post.title | escape }}</a> </h4>
    </header> 
  </article> -->
<article class="posts">
      <h4> &emsp;{{ post.date | date: "%b %d" }} &emsp;<a href="{{ post.url }}">{{ post.title | escape }}></a> </h4>
</article>

{% endfor %}
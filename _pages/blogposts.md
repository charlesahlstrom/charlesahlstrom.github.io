---
layout: posts
permalink: /blogposts/
title: "Blog posts"
author_profile: true
header:
  image: "/images/tree1.JPG"
  caption: "Branches"
---

{% for tag in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ tag | slugify }}" class="archive__subtitle">{{ tag }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}

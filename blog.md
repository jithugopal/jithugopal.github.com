---
layout: default
title: Blog | Jithu Gopal
permalink: /blog/
---

<ul class="post-list">
  {% for post in site.posts %}
    <li class="post-item">
      <span class="post-item-meta surplus">{{ post.date | date: "%b %-d, %Y" }}</span>

      <h2>
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </h2>
    </li>
  {% endfor %}
</ul>

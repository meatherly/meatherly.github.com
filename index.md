---
layout: page
---
{% include JB/setup %}
<div class="posts">
  {% for post in site.posts %}
    <div class="post">
      <div class="post-date">
        <span class="post-date-text">{{ post.date | date: "%B %-d, %Y" }}</span>
      </div>
      <div class="post-title">
        <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
      </div>
    </div>
  {% endfor %}
</div>






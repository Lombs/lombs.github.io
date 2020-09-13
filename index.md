---
layout: default
---
<h1>Latest Posts</h1>
<p>
  {% for post in site.posts %}
    <h2><a href="{{ post.url }}">{{ post.date}}: {{ post.title}}</a></h2>
  {% endfor %}
</p>




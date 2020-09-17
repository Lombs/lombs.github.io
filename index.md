---
layout: default
---
<h1>Latest Posts</h1>
<p>
  {% for post in site.posts %}
  {% assign currentdate = post.date | date: "%Y" %}
  {% if currentdate != date %}
    <h2 id="y{{currentdate}}">{{ currentdate }}</h2>
    {% assign date = currentdate %} 
  {% endif %}
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
  {% endfor %}
</p>




---
layout: default
---

[About me](./another-page.html)

My blog posts
{% for post in site.posts %}

{{ post.date | date:"%m/%d" }}
{{ post.date | date:"%Y" }}
{{ post.content | | split:'' | first }}
{{ post.date | date_to_string }}
{{ post.title }}
{% endfor %}




---
layout: default
---

{% for post in site.posts %}

{{ post.date | date:"%m/%d" }}
{{ post.date | date:"%Y" }}
{{ post.content | | split:'' | first }}
{{ post.date | date_to_string }}
{{ post.title }}
{% endfor %}

[Link to another page](./another-page.html).


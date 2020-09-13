---
layout: default
---

## About Me

{% include badge.html %}

{% if site.github.is_project_page %}
<p class="view"><a href="{{ site.github.repository_url }}">View the Project on GitHub <small>{{ site.github.repository_nwo }}</small></a></p>
{% endif %}

{% if site.github.is_user_page %}
<p class="view"><a href="{{ site.github.owner_url }}">View My GitHub Profile</a></p>
{% endif %}

{% if site.show_downloads %}
<ul class="downloads">
  <li><a href="{{ site.github.zip_url }}">Download <strong>ZIP File</strong></a></li>
  <li><a href="{{ site.github.tar_url }}">Download <strong>TAR Ball</strong></a></li>
  <li><a href="{{ site.github.repository_url }}">View On <strong>GitHub</strong></a></li>
</ul>
{% endif %}
<p class="view"><a href="{{ site.baseurl }}{% link index.md %}">Home</a></p>
<p class="view"><a href="{{ site.baseurl }}{% link sample.md %}">About SIEMphonies</a></p>
<p class="view"><a href="{{ site.baseurl }}{% link about.md %}">About Me</a></p>

[back](./)

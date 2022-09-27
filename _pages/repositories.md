---
layout: page
permalink: /repositories/
title: repositories
description: Some highlighted repositories that I maintain. I do most of my implementations in Rust, which is amazing for cryptographic protocols given the large amount of cryptography crates available and its strong safety properties.
nav: true
nav_order: 3
---

{% if site.data.repositories.github_repos %}
<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% for repo in site.data.repositories.github_repos %}
    {% include repository/repo.html repository=repo %}
  {% endfor %}
</div>
{% endif %}

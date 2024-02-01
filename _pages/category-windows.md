---
title: "Windows"
layout: archive
permalink: /Windows
author_profile: true
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.Windows %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
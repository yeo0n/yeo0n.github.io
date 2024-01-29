---
title: "Linux"
layout: archive
permalink: /Linux
author_profile: true
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.Linux %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}
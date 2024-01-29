---
title: "Android"
layout: archive
permalink: /Android
author_profile: true
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.Android %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}
---
title: "보안뉴스 스크랩"
layout: archive
permalink: categories/news
---

{% assign posts = site.categories.news %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
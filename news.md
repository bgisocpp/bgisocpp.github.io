---
layout: page
title: News
permalink: /news/
---

{% assign news_sorted = site.news | sort: "date" | reverse %}
{% for item in news_sorted %}
- **[{{ item.title }}]({{ item.url | relative_url }})** — {{ item.date | date: "%d/%m/%Y" }}{% if item.author %} · {{ item.author }}{% endif %}
{% endfor %}

{% unless news_sorted.size > 0 %}
No news yet.
{% endunless %}

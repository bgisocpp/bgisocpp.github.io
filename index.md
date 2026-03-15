---
layout: home
title: Home
---

## Upcoming Meetings

{% assign today = 'now' | date: "%Y-%m-%d" %}
{% assign upcoming = site.meetings | sort: "date" %}
{% assign has_upcoming = false %}

{% for meeting in upcoming %}
  {% assign meeting_date = meeting.date | date: "%Y-%m-%d" %}
  {% if meeting_date >= today %}
    {% assign has_upcoming = true %}
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}
  {% endif %}
{% endfor %}

{% unless has_upcoming %}
No upcoming meetings scheduled. Check back soon!
{% endunless %}

---

## About Us

We are the official Bulgarian mirror committee for ISO/IEC JTC 1/SC 22/WG 21 —
the international working group responsible for the C++ programming language standard.
Operating under the Bulgarian Standardization Institute (BDS), we contribute to the
evolution of the C++ standard and promote modern C++ practices within Bulgaria.

[Learn more about us →]({{ '/about/' | relative_url }})

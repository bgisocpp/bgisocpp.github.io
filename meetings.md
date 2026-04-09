---
layout: page
title: Meetings
permalink: /meetings/
---

{% assign today = site.time | date: "%Y%m%d" | plus: 0 %}

## Upcoming

{% assign has_upcoming = false %}
{% assign meetings_sorted_asc = site.meetings | sort: "date" %}
{% for meeting in meetings_sorted_asc %}
  {% assign meeting_date = meeting.date | date: "%Y%m%d" | plus: 0 %}
  {% if meeting_date >= today %}
    {% assign has_upcoming = true %}
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}
  {% endif %}
{% endfor %}

{% unless has_upcoming %}
No upcoming meetings scheduled.
{% endunless %}

## Past

{% assign has_past = false %}
{% assign meetings_sorted_desc = site.meetings | sort: "date" | reverse %}
{% for meeting in meetings_sorted_desc %}
  {% assign meeting_date = meeting.date | date: "%Y%m%d" | plus: 0 %}
  {% if meeting_date < today %}
    {% assign has_past = true %}
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}
  {% endif %}
{% endfor %}

{% unless has_past %}
No past meetings yet.
{% endunless %}

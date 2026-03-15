---
layout: page
title: Meetings
permalink: /meetings/
---

{% assign today = 'now' | date: "%Y-%m-%d" %}

## Upcoming

{% assign has_upcoming = false %}
{% for meeting in site.meetings %}
  {% assign meeting_date = meeting.date | date: "%Y-%m-%d" %}
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
{% assign meetings_sorted = site.meetings | sort: "date" | reverse %}
{% for meeting in meetings_sorted %}
  {% assign meeting_date = meeting.date | date: "%Y-%m-%d" %}
  {% if meeting_date < today %}
    {% assign has_past = true %}
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}
  {% endif %}
{% endfor %}

{% unless has_past %}
No past meetings yet.
{% endunless %}

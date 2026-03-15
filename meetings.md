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
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}{% if meeting.time %}, {{ meeting.time }}{% endif %}{% if meeting.location %} · {% if meeting.location_link %}[{{ meeting.location }}]({{ meeting.location_link }}){% else %}{{ meeting.location }}{% endif %}{% endif %}{% if meeting.presenter %} · Presenter: {{ meeting.presenter }}{% endif %}{% if meeting.sponsor %} · Sponsor: {{ meeting.sponsor }}{% endif %}
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
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}{% if meeting.location %} · {% if meeting.location_link %}[{{ meeting.location }}]({{ meeting.location_link }}){% else %}{{ meeting.location }}{% endif %}{% endif %}{% if meeting.presenter %} · Presenter: {{ meeting.presenter }}{% endif %}{% if meeting.sponsor %} · Sponsor: {{ meeting.sponsor }}{% endif %}
  {% endif %}
{% endfor %}

{% unless has_past %}
No past meetings yet.
{% endunless %}

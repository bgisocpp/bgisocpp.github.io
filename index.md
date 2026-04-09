---
layout: home
title: Home
---

## Upcoming Meetings

{% assign today = site.time | date: "%Y%m%d" | plus: 0 %}
{% assign sorted_meetings = site.meetings | sort: "date" %}
{% assign has_upcoming = false %}

{% for meeting in sorted_meetings %}
  {% assign meeting_date = meeting.date | date: "%Y%m%d" | plus: 0 %}
  {% if meeting_date >= today %}
    {% assign has_upcoming = true %}
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}
  {% endif %}
{% endfor %}

{% unless has_upcoming %}
No upcoming meetings scheduled. Check back soon!
{% endunless %}

---

## Past Meetings

{% assign has_past = false %}

{% for meeting in sorted_meetings reversed %}
  {% assign meeting_date = meeting.date | date: "%Y%m%d" | plus: 0 %}
  {% if meeting_date < today %}
    {% assign has_past = true %}
- **[{{ meeting.title }}]({{ meeting.url | relative_url }})** — {{ meeting.date | date: "%d/%m/%Y" }}
  {% endif %}
{% endfor %}

{% unless has_past %}
No past meetings yet.
{% endunless %}

---

## About Us

We are the official Bulgarian mirror committee for ISO/IEC JTC 1/SC 22/WG 21 —
the international working group responsible for the C++ programming language standard.
Operating under the Bulgarian Standardization Institute (BDS), we contribute to the
evolution of the C++ standard and promote modern C++ practices within Bulgaria.

Subscribe to our [mailing list](https://groups.google.com/g/bgisocpp-official) for announcements or follow us on
[LinkedIn](https://www.linkedin.com/company/bulgarian-c-working-group/).

[Learn more about us →]({{ '/about/' | relative_url }})

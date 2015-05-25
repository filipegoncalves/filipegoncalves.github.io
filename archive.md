---
layout: page
title: Archive
---

## Blog Posts

{% for post in site.posts %}
  *   {% assign d = post.date | date: "%-d" %}{% assign m = post.date | date: "%B" %}{% case m %}{% when 'April' or 'May' or 'June' or 'July' %}{{ m }} {% when 'September' %}Sept. {% else %}{{ post.date | date: "%b" }}. {% endcase %}{% case d %}{% when '1' or '21' or '31' %}{{ d }}st, {% when '2' or '22' %}{{ d }}nd, {% when '3' or '23' %}{{ d }}rd, {% else %}{{ d }}th, {% endcase %}{{ post.date | date: "%Y" }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

---
layout: home
title: CS XXXX
nav_exclude: true
permalink: /:path/
seo:
  type: Course
  name: CS XXXX
---

# CS XXXX, Fall 2026

Welcome to CS XXXX at Northeastern! This course page is intended as a "home page" for our course where you can find all of the resources we will use this term.

If you have any questions, feel free to reach out to Sherdil directly :)

## Calendar

**Note 1:** This is a *rough*, *in-progress* sketch of the semester, and things are subject to change. We can accurately predict the past, but predicting the future is hard!

**Note 2:** If you can't access any of these resources, make sure you're logged into your Northeastern Microsoft account. If you still have trouble, send me an email: I may need to manually add you.

{% for module in site.modules %}
{{ module }}
{% endfor %}


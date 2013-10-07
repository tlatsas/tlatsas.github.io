---
layout: page
title: "tags"
date: 2013-07-28 23:11
comments: false
sharing: false
footer: true
---
<ul>
{% for item in site.tags %}
    <li><a href="/blog/tags/{{ item[0] }}/">{{ item[0] | capitalize }}</a> [ {{ item[1].size }} ]</li>
{% endfor %}
</ul>

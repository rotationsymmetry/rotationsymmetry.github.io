---
layout: page
title: By Category
---

{% capture categories %}{% for category in site.categories %}{{ category | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign category = categories | split:',' | sort %}
{% for item in (0..site.categories.size) %}{% unless forloop.last %}
{% capture word %}{{ category[item] | strip_newlines }}{% endcapture %}
<h2 class="category" id="{{ word }}">{{ word }}</h2>
{% for post in site.categories[word] %}{% if post.title != null %}
* {{ post.date | date_to_string }} </span> [ {{ post.title }} ]({{ post.url }}) 
{% endif %}{% endfor %}
{% endunless %}{% endfor %}
<br/><br/>

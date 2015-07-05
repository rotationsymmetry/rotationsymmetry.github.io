---
layout: page
title: category
---

{% capture categories %}{% for category in site.categories %}{{ category | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign category = categories | split:',' | sort %}


{% for item in (0..site.categories.size) %}{% unless forloop.last %}
{% capture word %}{{ category[item] | strip_newlines }}{% endcapture %}
<article class="post">
<header class="post-header">
<h2 id="{{ word }}">{{ word }}</h2>
</header>
<ul>
{% for post in site.categories[word] %}{% if post.title != null %}

<li> {{ post.date | date_to_string }} <a href="{{ post.url }}">{{ post.title }}</a> </li>

{% endif %}{% endfor %}

</ul>
</article>
{% endunless %}{% endfor %}
<br/><br/>

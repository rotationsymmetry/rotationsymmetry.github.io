---
layout: page
---

{% for tag in site.tags%}

{% capture tg %}{{tag | first | strip_newlines}}{% endcapture %}

<article class="post">
<header class="post-header">
{% assign item=site.tag_names[tg]%}
{{item.name}}
</header>
<ul>
{% for post in site.tags[tg] %}
  <li><a href="{{ post.url }}/">{{ post.title }}</a></li>
{% endfor %}
</ul>
</article>
{% endfor %}

---
layout: page
---

{% for tag in site.tags%}

{% capture tg %}{{tag | first | strip_newlines}}{% endcapture %}

<article class="post">
<header class="post-header">
<h2 class="post-title"><a name="{{tg}}">{{site.tag_names[tg].name}}</a></h2>
</header>
<div class="Table">
{% for post in site.tags[tg] %}
<div class="Row">
  <div clas="Cell">{{post.date | date_to_string}}</div>
  <div class="Cell"><a href="{{ post.url }}">{{ post.title }}</a></div>
</div>
{% endfor %}
</div>
</article>
{% endfor %}

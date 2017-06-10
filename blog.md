---
layout: blog
title: Blog
permalink: /blog/
---

<h2 style="text-align:center;"> Things are happening all the time, all over the world! </h2>

<h6 style="text-align:center;">I may or may not write about some of them...</h6>

{% if site.categories.size > 0 %}
<h5 style="text-align:center;">
{% for category in site.categories %}
	<a href="{{site.url}}/categories/{{category[0]}}" style="font-size:{{category[1].size}}vw;">{{category[0]}}</a>
{% endfor %}
</h5>
{% endif %}
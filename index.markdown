---
layout: default
---

<div class="index-content">
<article class="content">
{% for post in site.categories.article limit: 1 %}
<header>
<h2 class="title"><a href="{{ post.url }}">{{ post.title }}</a></h2>
<div class="meta clearfix">
<div class="published">
<span class="caption">Posted:</span>
<span class="publishdate"><time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:'%B %d, %Y' }}</time></span>
</div>
{% if post.tags and post.tags.first %}
<div class="tags">
  <span class="caption">Tags:</span>
  {% for tag in post.tags %}
  <a href="/tags.html#{{ tag }}" title="{{ tag }}">#{{ tag }}</a>
  {% endfor %}
</div>
{% endif %}
</div>
</header>
<div class="entry-content">
{% if post.content contains '<!--more-->' %}
  {{ post.content  | split:'<!--more-->' | first }}
  <a  class="more more-link" href="{{ post.url }}"><i class="fa icon-right-open"></i>Continue reading..</a>
{% else %}
  {{ post.content }}
{% endif %}
</div>
{% endfor %}
</article>
<div class="divider"></div>
<ul class="listing" id="main-listing">
<li class="listing-seperator"><i class="fa icon-down-circled"></i><span class="sep">Earlier posts in this year..</span></li>
{% capture year %}{{ site.time | date:"%Y"}}{% endcapture %}
{% for post in site.categories.article offset:1 %}
{% capture y %}{{ post.date | date:"%Y"}}{% endcapture %}
{% if year != y %}
{% break %}
{% endif %}
<li class="listing-item">
  <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
  <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
</li>
{% endfor %}
<li class="listing-seperator"><a class="more" href="/archive.html"><i class="fa icon-right-hand"></i><span>Long long ago..</span></a></li>
</ul>
</div>

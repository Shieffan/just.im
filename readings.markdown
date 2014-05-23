---
title: My Readings
layout: page
---

<div id="reading-list">
{% for reading in site.categories.reading %}
<section class="reading">
    <h2  class="reading-title"><span><time datetime="{{ reading.date | date:"%Y-%m-%d" }}" title="{{ reading.date | date:"%Y-%m-%d" }}"><span class="y">{{ reading.date | date:'%Y' }}</span><span class="m">{{ reading.date | date:'%m' }}</span></time></span> {{ reading.title }}</h2>
    <div class="reading-content">
    {{ reading.content }}
    </div>
</section>
{% endfor %}
</div>

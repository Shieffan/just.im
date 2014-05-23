---
title: Tags
layout: page
---

<div id='tag_cloud'>
{% for tag in site.tags %}
<a href="#{{ tag[0] }}" data-id="{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{% if tag[0] contains ' ' %} `{{ tag[0] }}` {% else %} {{tag[0]}} {% endif %}</a>
{% endfor %}
</div>

<ul class="listing article-list">
{% for tag in site.tags %}
  <li class="tag-listing {{tag[0]}}" data-id="{{ tag[0] }}" id="{{ tag[0] }}"><i class="fa icon-quote-left-1"></i><span>{{ tag[0] }}</span>
<ul>
{% for post in tag[1] %}
  <li class="listing-item" data-id="{{ tag[0] }}">
  <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:'%Y-%m-%d' }}</time>
  <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>
</li>
{% endfor %}
</ul>
<script type="text/javascript">
document.addEventListener("DOMContentLoaded", function(event) {
    var tag_cloud = document.getElementById('tag_cloud');
    var tags = tag_cloud.getElementsByTagName('a');
    var min,max,size;
    for(var i=0,ii=tags.length;i<ii;i++){
        size = parseInt(tags[i].getAttribute('rel'));
        if(i==0){
            max = min = size;
            continue;
        }
        if(size<min){
            min=size;
        }
        if(size>max){
           max=size; 
        }
    }
    var ratio;
    for(i=0;i<ii;i++){
        size = parseInt(tags[i].getAttribute('rel'));
        ratio = (size-min)/(max-min);
        tags[i].style['font-size'] = 0.5+ratio*0.5+'em';
        tags[i].style.opacity = 0.6+ratio*0.4;
        tags[i].onclick=function(e){
            e.preventDefault();
            var list_id = this.getAttribute('data-id');
            document.location.hash = list_id; 
            showList(list_id);
        };
    } 
    function showList(list_id){
        var tag_listing = document.getElementsByClassName('tag-listing');
        for(var j=0;j<tag_listing.length;j++){
            var id = tag_listing[j].getAttribute('data-id');
            if(id == list_id){
                tag_listing[j].style.display = 'block'; 
            }else{
                tag_listing[j].style.display = 'none'; 
            }
        }
    }

    if(window.location.hash.length){
       showList(window.location.hash.split('#')[1]);
    }
});
</script>

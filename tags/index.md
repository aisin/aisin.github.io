---
title: tags
layout: home
---

<div class="mid-col">
    <div class="mid-col-container">
        <div id="content" class="inner">

			<div id='tag_cloud'>
			{% for tag in site.tags %}
			<a href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}</a>
			{% endfor %}
			</div>

			<ul class="listing">
			{% for tag in site.tags %}
				<li class="listing-seperator" id="{{ tag[0] }}">{{ tag[0] }}</li>
				{% for post in tag[1] %}
				<li class="listing-item">
				<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
				<a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
				</li>
				{% endfor %}
			{% endfor %}
			</ul>

		</div><!--/#content-->
    </div><!--/.mid-col-container-->
</div><!--/.mid-col-->


<script src="../js/jquery.tagcloud.js" type="text/javascript" charset="utf-8"></script> 
<script language="javascript">
$.fn.tagcloud.defaults = {
    size: {start: 1, end: 1, unit: 'em'},
      color: {start: '#a9d0f5', end: '#ff3333'}
};

$(function () {
    $('#tag_cloud a').tagcloud();
});
</script>
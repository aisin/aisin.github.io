---
layout : home
---

<div class="mid-col">
    <div class="mid-col-container">
        <div id="content" class="inner">
            {% for post in site.categories.blog %}
            <article class="post">
                <div class="meta">
                    <div class="date">
                        <time>{{ post.date | date:"%Y-%m-%d" }}</time>
                    </div>
                    <!--
                    <div class="tags">
                    {% if page.tags %}
                    tags:
                    {% for tag in page.tags %}
                    <a href="/tags/#{{ tag }}" title="{{ tag }}">{{ tag }}</a>&nbsp;
                    {% endfor %}
                    {% endif %}
                    </div>
                    -->
                </div>
                <h1 class="title"><a href="{{ post.url }}" itemprop="url">{{ post.title }}</a></h1>
                <div class="entry-content">{{ post.description }}</div>
            </article>
            {% endfor %}
            <!---->
            <div id="post-pagination" class="paginator">

              {% if paginator.total_pages > 1 %}
				  <nav id="pages_panel" role="pagination" data-total-pages="{{ paginator.total_pages }}" data-current-page="{{ paginator.page }}" data-total-posts="{{ paginator.total_posts }}">
					<div>
					{% if paginator.previous_page %}
					  <a class="page-prev" href="/{%if paginator.previous_page > 1 %}page/{{ paginator.previous_page }}/{% endif %}">&laquo; 上一页</a>
					{% endif %}

					{% for page in (1..paginator.total_pages) %}
					  {% if page == paginator.page %}
						<span class="page-num-current">{{ page }}</span>
					  {% else %}
						<a class="page-num" data-page="{{ page }}" href="/{%if page > 1 %}page/{{ page }}/{% endif %}">{{ page }}</a>
					  {% endif %}
					{% endfor %}

					{% if paginator.next_page %}
					  <a class="page-next" href="/page/{{ paginator.next_page }}/">下一页 &raquo;</a>
					{% endif %}
					  <br />
					  <span style="display: inline-block; float: right; font-size: 12px; color: #999;">
					  本博合计 {{ paginator.total_posts }} 篇文章，最后更新于 {{ site.time }}。
					  </span>
					</div>
				  </nav>
				{% endif %}
              (共{{ paginator.total_posts }}篇)
            </div><!---->
            <footer id="footer" class="inner">Copyright &copy; 2013 Aisin's Blog All Rights Reserved.</footer>
        </div><!--/#content-->
    </div><!--/.mid-col-container-->
</div><!--/.mid-col-->

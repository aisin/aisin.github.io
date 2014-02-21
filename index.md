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

              <nav id="pages">
				{% if paginator.previous_page %}
					{% if paginator.previous_page == 1 %}
						<a href="{{site.base_url}}/" class="previous">&lt;&lt; Vorige Seite</a>
					{% else %}
						<a href="{{site.base_url}}/page{{paginator.previous_page}}" class="previous">&lt;&lt; Vorige Seite</a>
					{% endif %}
				{% else %}
					<span class="previous">&lt;&lt; Vorige Seite</span>
				{% endif %}

				&bull;

				Seite {{paginator.page}} von {{paginator.total_pages}} (insgesamt {{ paginator.total_posts }} Posts)

				&bull;

				{% if paginator.next_page %}
					<a href="{{site.base_url}}/page{{paginator.next_page}}" class="next">Nächste Seite &gt;&gt;</a>
				{% else %}
					<span class="previous">Nächste Seite &gt;&gt;</span>
				{% endif %}
				</nav>
              (共{{ paginator.total_posts }}篇)
            </div><!---->
            <footer id="footer" class="inner">Copyright &copy; 2013 Aisin's Blog All Rights Reserved.</footer>
        </div><!--/#content-->
    </div><!--/.mid-col-container-->
</div><!--/.mid-col-->

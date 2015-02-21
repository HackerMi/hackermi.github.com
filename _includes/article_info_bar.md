<div class="post-meta">
    {% assign page = include.cur_page %}
	<span>
		发布时间:
		<a href="{{ page.url }}">{{ page.date | date:"%Y年%m月%d日" }}</a>
	</span>
	<span><i class="fa fa-ellipsis-v"></i></span>
	<span>
		标签:
		{% for tag in page.tags %}
		<a href="/tags.html#{{ tag }}-ref">{{ tag }}</a>
		{% endfor %}
	</span>
	{% unless page.title contains '【转载】' %}
	<span><i class="fa fa-ellipsis-v"></i></span>
	<span>
	   版权信息: 【自由转载-非商用-非衍生-<a href="/about.html">保持署名</a>】
	</span>
	{% endunless %}
</div>
<div class="post-meta">
	<span>
		发布时间:
		<a href="{{ include.cur_page.url }}">{{ include.cur_page.date | date:"%Y年%m月%d日" }}</a>
	</span>
	<span><i class="fa fa-ellipsis-v"></i></span>
	<span>
		标签:
		{% for tag in include.cur_page.tags %}
		<a href="/tags.html#{{ tag }}-ref">{{ tag }}</a>
		{% endfor %}
	</span>
	{% unless include.cur_page.title contains '【转载】' %}
	<span><i class="fa fa-ellipsis-v"></i></span>
	<span>
	   版权信息: 【自由转载-非商用-非衍生-<a href="/about.html">保持署名</a>】
	</span>
	{% endunless %}
</div>
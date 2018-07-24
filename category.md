---
layout: default
title: categories
---

<div class="post">
    <h1 class="pageTitle">categories</h1>
	<ul>
		<div><a href="./category/컴퓨터공학">컴퓨터공학</a>
			{% for post in site.categories.컴퓨터공학 %}
				<li><a href="{{ post.url }}">{{ post.title }}</a></li>
			{% endfor %}
		</div>
		<br>
		<div><a href="./category/Python">Python</a>
			{% for post in site.categories.Python %}
				<li><a href="{{ post.url }}">{{ post.title }}</a></li>
			{% endfor %}
		</div>
		<br>
		<div><a href="./category/Django">Django</a>
			{% for post in site.categories.Django %}
				<li><a href="{{ post.url }}">{{ post.title }}</a></li>
			{% endfor %}
		</div>
		<br>
		<div><a href="./category/Web">Web</a>
			{% for post in site.categories.Web %}
				<li><a href="{{ post.url }}">{{ post.title }}</a></li>
			{% endfor %}
		</div>
	</ul>
</div>
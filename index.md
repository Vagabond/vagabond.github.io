---
layout: page
---
{% include JB/setup %}

{% for post in site.posts limit:3 %}
<h3><a href="{{ post.url }}">{{ post.title }}</a> - <abbr>{{ post.date | date_to_string }}</abbr></h3>
{{ post.content }}
<hr style="width: 40%;"/>
{% endfor %}

<hr style="width: 40%;"/>
<a href="{{ BASE_PATH }}{{ site.JB.archive_path }}">Archive</a>

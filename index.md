{% for post in site.posts %}- [{{ post.title }}]({{ post.url }})<br/><small>{{ post.date | date: "%B %e, %Y" }}</small>
{% endfor %}	

# Java

{% for page in pages %}
{% if "Java" in page.meta.tags %}
* [{{ page.title }}]({{ page.url }})
{% endif %} 
{% endfor %}

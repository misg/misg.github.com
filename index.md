---
layout: page
---
{% include JB/setup %}

<div class="blog-index">
    {% for post in site.posts limit:3 %}
        <h1 class="entry-title">
            {% if post.title %}
                <a href="{{ root_url }}{{ post.url }}">{{ post.title }}</a>
            {% endif %}
        </h1>

        <div class="entry-content">{{ post.content }}</div>
    {% endfor %}
</div>


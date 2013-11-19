---
layout: page
title:
tagline:
---
<div class="row">
    {% for post in site.posts limit:3 %}
    <div class="span4">
        <div class="index-post-title">
            <a href="{{ BASE_PATH }}{{ post.url }}"><h2>{{ post.title }}</h2></a>
        </div>
        <hr />
        <p>{% if post.thumbnail %}</p>
        <img src="{{ post.thumbnail }}" style="height: 280px;" align="center"/>
        {% else %}
        <img src="/images/nothumbnail.png" style="height: 280px;" align="center"/>
        {% endif %}
        <p>&nbsp;</p>
        <div>
            {{ post.content | strip_html | truncatewords:20 }}
        </div>
        <p>
            <a class="btn" href="{{ BASE_PATH }}{{ post.url }}">阅读全文...</a>
        </p>
    </div>
    {% endfor %}
</div>

{% for post in site.posts limit:15 offset:3 %}
<hr />
<div class="row">
    <div class="span2">
        {% if post.thumbnail %}
            <img src="{{ post.thumbnail }}" align="center" />
        {% else %}
            <img src="/images/nothumbnail.png" align="center"/>
        {% endif %}
    </div>
    <div class="span10">
        <p>
            <a href="{{ BASE_PATH }}{{ post.url }}">
                <h3>{{ post.title }}</h3>
            </a>
        </p>
        <p>{{ post.content | strip_html | truncatewords:40 }}</p>
    </div>
</div>
{% endfor %}

---
layout: index
title:
tagline:
---
<div class="row">
        {% for post in site.posts limit:3 %}
        <div class="span4">
            <div class="index-post-title">
                <a href="{{ BASE_PATH }}{{ post.url }}">
                    <h2 class="index-title">{{ post.title }}</h2>
                </a>
            </div>
            <p>{% if post.thumbnail %}</p>
            <div class="thumbnail-wrapper">
                <img src="{{ post.thumbnail }}" />
            </div>
            {% else %}
            <div class="thumbnail-wrapper">
                <img src="/images/nothumbnail.jpg" />
            </div>
            {% endif %}
            <p>&nbsp;</p>
            <div class="brief-content">
                {{ post.content | strip_html | truncate:50 }}
            </div>
            <p>
                <a class="btn" href="{{ BASE_PATH }}{{ post.url }}">阅读全文...</a>
            </p>
        </div>
        {% endfor %}
</div>
<div class="row">
<div class="span9">
    {% for post in site.posts offset:3 limit:8 %}
    <hr class="content-hr"/>
    <div class="row content-row">
        <div class="span2">
            {% if post.thumbnail %}
            <div class="thumbnail-wrapper-mini">
                <img src="{{ post.thumbnail }}" />
            </div>
            {% else %}
            <div class="thumbnail-wrapper-mini">
                <img src="/images/nothumbnail.jpg" />
            </div>
            {% endif %}
        </div>
        <div class="span10">
            <p>
                <a href="{{ BASE_PATH }}{{ post.url }}">
                    <div class="index-title">{{ post.title }}</div>
                </a>
            </p>
            <p>{{ post.content | strip_html | truncate:150 }}</p>
        </div>
    </div>
    {% endfor %}
</div>
<div class="span3">
    <div class="index-sidebar">
        <div class="sidebar-title">文章分类</div>
        <div class="column-sidebar">
            {% for cat in site.categories %}
            <ul>
                <li class="column-list-item">
                    <a href="/categories.html">{{ cat[0] }}</a>
                </li>
            </ul>
            {% endfor %}
        </div>
    </div>
    <div class="index-sidebar">
        <div class="sidebar-title">标签分类</div>
        <div class="column-sidebar">
            {% for tag in site.tags %}
            <ul>
                <li class="column-list-item">
                    <a href="/tags.html">{{ tag[0] }} </a>
                </li>
            </ul>
            {% endfor %}
        </div>
    </div>
</div>
</div>

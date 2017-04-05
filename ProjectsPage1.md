---
layout: page
title: Projects 
permalink: /page/2
backgrounds: 
    - https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/backgrounds/368266-colorful-flower-field.jpg

---
<center> <a href="https://www.icloud.com/sharedalbum/#B0g5yeZFhGZWmvT"> Art Projects and Works in Progress </a> </center>
<div class="home">
    <ul class="post-list">
        {% if site.pagination %}
        {% assign source = site.categories.projects %}
        {% endif %}

        {% for post in source %}
        <li>
            <a href="{{ post.url | prepend: site.baseurl }}" class="post-list-link">
                <div class="post-list-info">
                    <h2 class="post-list-title">{{ post.title }}</h2>
                    <span class="post-list-meta">{{ post.date | date: "%B %-d, %Y" }}</span>
                </div>
            </a>

            <img src="{{ post.thumb }}" class="post-list-thumb" />
        </li>
        {% endfor %}
    </ul>

    <!-- <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p> -->
    <!-- <p class="rss-subscribe">
        <a href="{{ "/feed.xml" | prepend: site.baseurl }}">
            Subscribe via RSS
            <i class="icon ion-social-rss"></i>
        </a>
    </p> -->
</div>

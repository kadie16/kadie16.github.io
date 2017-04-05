---
layout: page
title: KD at S.E.A.
permalink: /page/4
backgrounds:
    - https://raw.githubusercontent.com/kadie16/kadie16.github.io/master/assets/images/backgrounds/asiaMap.jpg
---
<h1> <center>A blog about my summer in South East Asia </h1></center>

    I spent last summer living and working in Singapore. During the week, I worked on a 3D viewer project at the A*STAR Institute of High Perfomance Computing. On the weekends, I traveled as much as I could! <br> <br> <center> <a href="https://www.icloud.com/sharedalbum/#B03Grq0zw6DhcW"> Photostream </a> </center> <br> A couple of these posts are unfinished, but I decided to share them here anyway since it is unlikely I will get around to polishing them. 
        
<div class="home"> 
    <ul class="post-list">
        {% if site.pagination %}
        {% assign source = site.categories.abroad %}
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

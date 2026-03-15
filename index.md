---
layout: default
title: Welcome
whereami: index
---
### About the site by github io jekyll
記錄一些資訊 .... 將 http://echochio.pixnet.net/blog 一些來紀錄到此

做技術 聯繫我： [<span style="color:#000000">chio@echochio.dpdns.org</span>]

好記性不如爛筆記， 我有 30 多年了技術，使用和學習過很多技術，但在長時間不使用後，很多東西就忘掉了，來記錄一些筆記。

現在 用 git 架設的網站，及git 紀錄文件 ，也用 cloudflare 的轉信，cloudflare 的 tunnel 測設機器。

---

#### 文章列表:

<div class="post-list-body">
    <div post-cate="All">
        <ul>
            {% for post in site.posts %}
            <li>{{ post.date | date:"%Y-%m-%d" }} <a href="{{ post.url }}"> {{ post.title }}</a></li>
            {% endfor %}
        </ul>
    </div>
    {% for tag in site.tags %}
      <div post-cate="{{tag | first}}">
        <ul>
        {% for posts in tag  %}
          {% for post in posts %}
            {% if post.url %}
              <li>{{ post.date | date:"%Y-%m-%d" }} <a href="{{ post.url }}"> {{ post.title }}</a></li>
            {% endif %}
          {% endfor %}
        {% endfor %}
        </ul>
      </div>
    {% endfor %}
</div>

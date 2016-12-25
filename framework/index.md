---
layout: home
---
<div class="index-content framework">
    <div class="section">
    <ul class="artical-cate">
            <li style="text-align:left"><a href="/"><span>笔记</span></a></li>
            <li style="text-align:center"><a href="/opinion"><span>随笔</span></a></li>
            <li class="on" style="text-align:center"><a href="/framework"><span>框架</span></a></li>
            <li style="text-align:right"><a href="/translate"><span>翻译</span></a></li>
        </ul>
    <div class="cate-bar"><span id="cateBar"></span></div>
        <ul class="artical-list">{% for post in site.categories.framework%}
            <li>
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>{% endfor %}
        </ul>
    </div>
    <div class="aside"></div>
</div>
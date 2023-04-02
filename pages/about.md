---
layout: page
title: About
description: About Jesse and this website.
keywords: Jesse, website
comments: true
menu: 关于
permalink: /about/
---

在这里，记录一切探索过程中的灵感与新知。

这里是Jesse aka 白泽，本科在读英专生，喜欢数码与科技，同时也吸点Furry。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'mazhuang.org' %}
<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ site.url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
</li>
{% endif %}
</ul>


## 兴趣分布

{% for interest in site.data.interests %}
### {{ interest.name }}
<div class="btn-inline">
{% for keyword in interest.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

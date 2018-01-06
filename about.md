---
layout: page
title: 关于
menu: About
---
{% assign current_year = site.time | date: '%Y' %}

super_fazai
===
男 90后

## 概况

- 邮箱：superonesfazai@gmail.com
- 主页：[http://superonesfazai.github.io](http://superonesfazai.github.io)
- twitter：[@twitter](http://twitter.com)

计算机专业毕业，{{ current_year | minus: 2015 }} 年在职工作经验，{{ current_year | minus: 2015 }} 年 爬虫 开发经验。

## keywords
<div class="btn-inline">
{% for keyword in site.skill_keywords %} <button class="btn btn-outline" type="button">{{ keyword }}</button> {% endfor %}
</div>

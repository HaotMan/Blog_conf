---
title: Hexo页面响应优化
categories: hexo
tags: 
  - hexo
permalink: hexo_opt
date: 2020/6/1 20:46:25

---

在hexo服务器配置好主题和博客文章后，发现主页访问速度极慢，最长可达到十几秒。

用Chrome分析一下页面加载项，发现加载的短板是谷歌字体和jquery库

<!--more--> 

![hexo_load_analysis](./images/hexo_load_analysis.png)

于是将其换位国内的CDN服务

修改文件
{% blockquote %}
hexo目录/themes/主题名称/layout/_partial/head.ejs
{% endblockquote %}

将`fonts.googleapis.com`替换为国内科大的库`https://fonts.lug.ustc.edu.cn/`或捷速网络`https://fonts.css.network/ `

注：360的提供的节点fonts.useso.com 已停止使用


JQuery替换：

{% blockquote %}
hexo目录/themes/主题名称/layout/_partial/after-footer.ejs
{% endblockquote %}

将`//ajax.googleapis.com/ajax/libs/jquery/2.0.3/jquery.min.js `替换为`//code.jquery.com/jquery-2.0.3.min.js `


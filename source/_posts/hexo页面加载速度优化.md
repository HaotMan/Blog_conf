---
title: Hexo页面响应优化
categories: hexo
tags: 
  - hexo
permalink: hexo_opt
date: 2020/6/3 20:41:52

---

在hexo服务器配置好主题和博客文章后，发现主页访问速度极慢，最长可达到十几秒。

用Chrome分析一下页面加载项，发现加载的短板是谷歌字体和jquery库

<!--more--> 

![hexo_load_analysis](https://img-blog.csdnimg.cn/20200608005226404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01TRE5fdGFuZw==,size_16,color_FFFFFF,t_70#pic_center)

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



> 另外，可以下载懒加载插件并在配置文件中配置图片lazyload，进一步减少网页呈现的时间
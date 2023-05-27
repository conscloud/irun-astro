---
layout: '../../layouts/MarkdownPost.astro'
title: 'Astro集成cusdis评论系统'
pubDate: 2023-05-27
description: '在本站集成cusdis评论系统，支持留言评论'
author: 'ZhJy'
cover:
    url: ''
    square: ''
    alt: 'cover'
tags: ["分享","Astro"] 
theme: 'light'
featured: true

meta:
 - name: author
   content: ZhJy
 - name: keywords
   content: key3, key4

keywords: Astro, 留言, 评论!
---

**感谢：**[天真的和伤感的梦想家 (loongphy.com)](https://blog.loongphy.com/)，使用[Loongphy/blog: 个人博客 (github.com)](https://github.com/Loongphy/blog)的源代码。

### 前言

根据[austin2035/astro-air-blog: A minimalist, beautiful, responsive blogging program written in Astro.一个简约、漂亮并且支持响应式的博客程序，基于 Astro 构建。](https://github.com/austin2035/astro-air-blog)完成本站的搭建，但是没有评论功能总觉得少了点什么，在评价区找到了相关方案，集成Cusdis。

### 注册Cusdis账号

1. 在[Cusdis - Lightweight, privacy-first, open-source comment system](https://cusdis.com/)注册一个账号，可以使用github账号直接登录。
2. 添加网站地址

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305271626798.png" style="zoom:50%;" />

3. 点击`Embed Code`,复制代码

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305271627366.png" style="zoom:50%;" />

### 修改代码

修改`MarkdownPost.astro`代码，在后面加上下列代码：

~~~js
  <!-- 集成cusdis评论系统 -->
    <div class="component">      
        <div id="cusdis_thread"
        data-host="https://cusdis.com"
        data-app-id="XXX-XXX-XXX"
        data-page-id={ Astro.url }
        data-page-url={ Astro.url }
        data-page-title={ title }
      ></div>
    </div>
    <!-- 评论功能结束    -->   

    <Footer />
    <script is:inline>
      var script = document.createElement("script");
      script.src = "/static/js/initPost.js";
      document.head.appendChild(script);
    </script>
    <!-- 查看评论留言 -->
    <script defer src="https://cusdis.loongphy.com/js/widget/lang/zh-cn.js"
    ></script>
    <script async defer src="https://cusdis.loongphy.com/js/cusdis.es.js"
    ></script>
    <script
      defer
      data-host="https://cusdis.com"
      data-app-id="XXX-XXX-XXX"
      src="https://cusdis.loongphy.com/js/cusdis-count.umd.js"
    ></script>
    <!-- 留言功能结束 -->
~~~

其中zh-cn.js、cusdis.es.js及cusdis-count.umd.js三个文件直接使用的[天真的和伤感的梦想家 (loongphy.com)](https://blog.loongphy.com/)的

保存提交后，刷新页面就已经出现评论功能了：

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305271634898.png" alt="评论功能" style="zoom:50%;" />

提交的评价可以在cusdis的控制台审核、回复、删除：

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305271636107.png" alt="评论操作" style="zoom:50%;" />

审核通过后，可以在文章后面查看：

<img src="https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305271639633.png" alt="评论查看" style="zoom:50%;" />

---
title: Next 主题配置
comments: true
date: 2019-01-26 00:02:45
categories:
- 工作环境配置
tags:
- Next
---

<!--more-->
https://donlex.cn/archives/55e73569.html

### 修改标签效果
修改模板
`/themes/next/layout/_macro/post.swig`，搜索 `rel="tag">#`，将 `#` 换成`<i class="fa fa-tag"></i>`

### 网站底部字数统计
安装
`$ npm install hexo-wordcount --save`

然后在`/themes/next/layout/_partials/footer.swig`文件尾部加上：
```html
<div class="theme-info">
  <div class="powered-by"></div>
  <span class="post-count">博客全站共{{ totalcount(site) }}字</span>
</div>
```

---
title: hexo br 问题
date: 2016-12-14 17:44:10
tags:
- hexo
---

这两天用vim编写博客，在vim编写时使用了换行符，而且在hexo中同样也会显示出来，困扰
了好久。按照markdown的语法来说这应该不会出现这个问题。

查阅了一些信息，hexo博客是由`hexo-renderer-marked`(package.json中含有)解析的，然
后找到这一个依赖源在博客目录下`node_modules/hexo-renderer-marked/`,修改文件
`index.js`找到 `breaks` 改为false。即可
```javascript
hexo.config.marked = assign({
  gfm: true,
  pedantic: false,
  sanitize: false,
  tables: true,
  breaks: false,
  smartLists: true,
  smartypants: true
}, hexo.config.marked);
```
参考网址: [链接
1](http://www.bleachlei.site/blog/2016/11/02/Hexo-br-%E8%BF%87%E5%A4%9A%E9%97%AE%E9%A2%98/)
[链接2](http://ranler.github.io/2013/12/31/hexo-breaks/)

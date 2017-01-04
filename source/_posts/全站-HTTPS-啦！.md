---
title: 全站 HTTPS 啦！
date: 2016-12-28 11:38:22
categories: 技术
tags: Https
---
主要解决的两个问题：
## Hexo Next 主题菜单链接
Next 的菜单链接有个问题，如果在 github pages 部署用的第三方域名（默认域名用户可能没有这个问题），首页是 https 的，点击菜单后会变成 http，我看了下请求发现请求确实是 https 但是会 301 成 http，我就费解了，最后在这个 [issue](https://github.com/iissnan/hexo-theme-next/issues/1187#issuecomment-257788310) 里发现了答案，在链接最后加上 '/' 即可

## CloudFlare 设置 Page Rules
设置方法：Page Rules -> Create Page rule -> Add a Setting -> Always Use HTTPS
然后把 http://xxx.com/* 添加进去，这样就会强制 301 到 https 上了

哦，今天还把主题里的 Google Fonts 去掉了，还装了 [hexo-all-minifier](https://github.com/chenzhutian/hexo-all-minifier) 插件，打开速度大幅提升

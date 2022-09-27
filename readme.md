# 个人博客

博客使用jekyll和gitpage构建和部署，推送新博文后，gitpage会自动构建新的页面

## 添加新的blog

博客文件放置于_post目录下，文件名称使用YYYY-MM-DD-NAME-OF-POST.md格式，文件头部添加如下内容。

```
---
layout: post
title: "POST TITLE"
date: YYYY-MM-DD hh:mm:ss -0000
categories: CATEGORY-1 CATEGORY-2
---
```

## 本地测试

bundle install
bundle exec jekyll serve

## 相关资料

[GitPages和Jekyll配置指南](https://docs.github.com/cn/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll)

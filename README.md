# renrenche-fe.github.io
团队博客，使用 [hexo](https://hexo.io/zh-cn/) 搭建，主题为 [material](https://material.viosey.com/)

## 访问
[renrenche-fe.github.io](https://renrenche-fe.github.io/)

## 写文章
```
hexo new [layout] <title>
```
e.g. `hexo new my-first-article`，将生成 `/source/_posts/my-first-article.md` 文件

### 设置 author [Require]
打开 `my-first-article` 文件，添加 author 字段
```
---
title: 论前端工程师的自我修养
date: 2017-07-15 20:43:19
tags: [成长]
author: your_username
---
```
文章发表后将默认使用 `your_username` 作为作者名称

### 设置头像 [Require]
在 `/source/img` 下创建 `your_username` 文件夹，在文件夹下创建 `avatar.png` 图片，这个图片将作为你的头像

### 设置昵称
编辑 `/_config.yml`，在 `authors:` 下添加 `your_username: your_nickname` 键值对，文章发表后将使用 `your_nickname` 作为作者名称

## 本地预览
```
hexo s
```

## 生成页面
```
hexo g
```

## 部署
```
hexo d
```

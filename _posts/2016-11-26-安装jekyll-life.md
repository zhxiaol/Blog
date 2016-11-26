---
layout: post
title: 如何在Mac下安装jekyll
tags:
- Jekyll
- life
categories: life
description: Jacman 是一款扁平化，有着响应式设计的 Jekyll 主题。本站正式使用了 Jacman 主题。Jacman 基于 Jacman 的 Hexo 主题修改而来。你可以前往本站和 Demo 预览更多关于本主题的更多效果。如果你有任何问题或意见欢迎到 GitHub 发表 issue。
---
##主题介绍
好坑，只是想搭建个简单的blog用来记录学习，没想到填了一天的坑。技术真的就像婚前和婚后，回头再看，其实也没啥。

<!-- more -->
##配置指南
###更新gem
```
sudo gem update --system
```
###安装jekyll
```
sudo gem install jekyll

安装成功后，查看版本
jekyll -v

遇到的坑，如果安装失败 可以尝试安装到 /usr/local/bin/目录下
sudo gem install -n /usr/local/bin/ jekyll
```
###使用jekyll
```
生成博客
jekyll build

开启本地预览
jekyll server

这块遇到的坑最深，差点没爬出来
如果是缺少依赖直接
sudo gem install xxx

装不上就
sudo gem install -n /usr/local/bin/ xxx

要求装固定如1.1.0版本就
sudo gem install -v 1.1.0 xxx

最后要是有多个版本的依赖 可以选择删除一个 或者用
bundle exec jekyll build
bundle exec jekyll server
```

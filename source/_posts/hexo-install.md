---
tags: hexo
categories: 前端
title: 搭建github博客-hexo安装与使用
date: 2016-10-22
---
用hexo搭建一个博客，记录一下。
> 转自 [Jekyll迁移到Hexo搭建个人博客](http://www.ezlippi.com/blog/2016/02/jekyll-to-hexo.html?utm_source=tuicool&utm_medium=referral)

个人博客以前由jekyll搭建，主要问题是目录、Rss、sitemap无法自动生成，根据DRY的原则在网上找了下答案，最终发现了用Hexo来搭建博客的方法，配置完之后一劳永逸，目录、Rss和sitemap都是自动生成，解决了我之前的困惑。

# 从Jekyll迁移到Hexo

## 安装Hexo

安装
```
mkdir hexo  #创建一个文件夹
cd hexo
npm install -g hexo-cli
npm install hexo --save
```

部署Hexo：在Git shell 中输入
```
hexo init
```

<!-- more -->

安装Hexo 插件：自动生成sitemap,Rss，部署到git等，建议安装
```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
```

# Hexo常用的几个命令

## 创建新博文

执行new命令，生成指定名称的文章至hexo\source_posts\postName.md。
```
hexo new [layout] "postName" #新建文章
```
其中layout是可选参数，默认值为post。有哪些layout呢，请到scaffolds目录下查看，这些文件名称就是layout名称。当然你可以添加自己的layout，方法就是添加一个文件即可，同时你也可以编辑现有的layout，比如post的layout默认是hexo\scaffolds\post.md
```
title: { { title } }
date: { { date } }
tags:
---
```
> 请注意，大括号与大括号之间我多加了个空格，否则会被转义，不能正常显示。

我想添加categories，以免每次手工输入，只需要修改这个文件添加一行，如下：
```
title: { { title } }
date: { { date } }
categories: 
tags: 
---
```
更多信息参考：[Writing](https://hexo.io/docs/writing.html)

## 运行服务器

```
$ hexo server
```

## 生成静态站点文件

```
$ hexo generate
```

## 部署到Git
部署到Github前需要配置_config.yml文件

添加如下内容：
```
deploy:
	type: git
	repository: git@github.com:EZLippi/EZLippi.github.io.git
	branch: master
```
然后输入：
```
$ hexo deploy
```

# 主题设置

本博客采用了iissnan的Next主题，他的博客有详细的安装教程，这里贴下链接[next](http://theme-next.iissnan.com/)，有需要的朋友直接参考他写的教程，一气呵成~

## 下载主题

```
$ cd hexo目录
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

## 应用Hexo主题

在hexo目录下找到_config.yml配置文件，找到 theme 字段，并将其值更改为 next，如下所示：
```
theme: next
```

更多设置参考[next主题](http://theme-next.iissnan.com/)


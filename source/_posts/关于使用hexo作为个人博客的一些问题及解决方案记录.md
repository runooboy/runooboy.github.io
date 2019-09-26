---
title: 关于使用hexo作为个人博客的一些问题及解决方案记录
date: 2019-07-01 14:48:09
category:
- tools
tags:
- hexo
---
本文主要记录使用hexo作为个人博客时遇到的常见问题以及解决方案，还有别的问题就未完待续咯~
<!-- more -->

### hexo中文章分类问题
首先必须new一个page categories：
``` bash
$ hexo new page categories
```
然后这个时候修改categories下的index.md文件，主要修改type：
``` bash
title: categories
type: "categories"
```
此时，文章分类问题已经基本解决，接下来只需要在写文章的时候加入category即可，比如：
``` bash
category:
- 人工智能
```

### hexo中图片显示问题
对于hexo中的图片显示问题，首先修改hexo的_config.yml文件中的post_asset_folder，将这一项改为true：
``` bash
post_asset_folder: true
```
接下来使用hexo的new新建文章会带上一个相同名称的文件夹，在这个文件夹下可以存放该文章需要的图片，然后在文章中使用
``` bash
![](image.png)
```
即可在文章中看到该图片。

### hexo更换主题
这种更换主题的问题在百度上有许多，个人倾向使用next主题，直接百度hexo next进入官网，下载next主题将其放入themes下，然后在配置文件_config.yml中修改theme为：
``` bash
theme: next
```
然后使用hexo clean清楚缓存，重新使用hexo g生产静态页面即可刷新。

### hexo d后出现ERROR Deployer not found: git报错
这个问题直接使用下面的命令即可：
``` bash
npm install --save hexo-deployer-git
```
详细可以参考 [next官方文档](http://theme-next.iissnan.com/)

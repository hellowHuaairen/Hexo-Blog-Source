---
title: Next主题中添加超级好玩的看板娘（CDN方式）
date: 2019-08-18 10:48:56
tags:
    Hexo Next
categories:
    Hexo_Next博客搭建
---

本篇文章主要介绍如何向基于Hexo的博客中添加一个超级好玩的看板娘。
<!--more-->
# 1. Fork看板娘项目到自己的Github

Hexo博客本身是支持看板娘的，但是原本的只有一个人物，一点也不好玩，stevenjoezhang大佬写了一个可以说话，可以换装的看板娘项目并开源，使用起来也是非常的方便。

首先访问 [live2d-widget](https://github.com/stevenjoezhang/live2d-widget) 的 Github 仓库，点击右上角 `Fork` 该项目到自己的仓库：

![mark](http://mculover666.cn/image/20190818/bi2Pxy8hFWjS.png?imageslim)

![mark](http://mculover666.cn/image/20190818/tSMVIoJvovUh.png?imageslim)

# 2. 向 Next 中添加看板娘

修改 `/themes/hexo-theme-next/layout/_layout.swing` 文件，在最后添加下面这行代码：
```xml
<script src="https://cdn.jsdelivr.net/gh/Mculover666/live2d-widget/autoload.js"></script>
```

>注意：请将CDN链接中的 `Mculover666` 换成你的用户名！

# 3. 使能看板娘

编辑**主题配置文件** `/themes/hexo-theme-next/_config.yml`，添加如下内容：
```xml
# ---------------------------------------------------------------
# 自定义看板娘
# ---------------------------------------------------------------
live2d:
  enable: true
```

这个时候重新生成并部署博客，就可以看到博客左下角出现一个看板娘了哈哈。

![mark](http://mculover666.cn/image/20190818/mSQqKusaFAdr.gif)

**这个看板娘是默认的，在左下角，如果想要修改它的位置，大小，说话内容，该如何办呢？**

# 4. 定制一个属于你自己的看板娘

第 1 步中我们fork了看板娘项目到自己的 Github 中，然后使用CDN方式部署到了Hexo博客中，接下来讲述如何进行修改，定制一个属于你自己的看板娘~

## clone 你的项目到本地
将项目 clone 到本地：
```bash
git clone https://github.com/Mculover666/live2d-widget.git
```

>注意是 clone 你自己的 Github 中的看板娘项目!

## 修改文件

首先说明一下看板娘项目中几个文件的作用：

- `autoload.js`：自动加载看板娘；
- `waifu.css`：看板娘样式；
- `waifu-tips.js`：看板娘说话的脚本；
- `waifu-tips.json`：看板娘说话的内容；

### 修改加载看板娘的路径（必须）
在 `autoload.js` 的开头定义了加载看板娘的路径，注意这里需要使用绝对路径：
```
//注意：live2d_path参数应使用绝对路径
const live2d_path = "https://cdn.jsdelivr.net/gh/Mculover666/live2d-widget/";
//const live2d_path = "/live2d-widget/";
```

> 注意：将CDN路径中的 `Mculover666` 改为你自己的Github用户名。

### 修改看板娘布局和样式
在 `waifu.css` 中修改：

![mark](http://mculover666.cn/image/20190818/Wi9MnvXbm9SY.png?imageslim)

![mark](http://mculover666.cn/image/20190818/KvFmpk5rszHi.png?imageslim)

更多的修改可以自行修改哦~

### 暂存提交推送三连

修改完成后，将修改暂存，然后提交，最后**一定要推送到远程库**：
```bash
git status
git add -A
git commit -m "modify position to right"
git push origin master
```
### 发布新版本
我们是使用了Github免费的CDN服务，所以还需要发布一个新的版本：

![mark](http://mculover666.cn/image/20190818/Op2QmULLAWJ6.png?imageslim)

![mark](http://mculover666.cn/image/20190818/SsnBefsig3Pa.png?imageslim)

![mark](http://mculover666.cn/image/20190818/kvMsAcV9wc90.png?imageslim)

### 修改CDN链接
Github上CDN的使用方式为：
```bash
https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@发布的版本号/文件路径
```
所以在博客中的`/themes/hexo-theme-next/layout/_layout.swing`文件中修改CDN链接使用最新的版本：
```xml
<script src="https://cdn.jsdelivr.net/gh/Mculover666/live2d-widget@1.0.0/autoload.js"></script>
```

### 重新生成部署博客
```bash
hexo clean
hexo g
hexo d
```
来看看效果哈哈哈，看板娘已经成功的跑到了右边：

![mark](http://mculover666.cn/image/20190818/tYQyMphBy51z.png?imageslim)

**<font color="#FF0000">更多精彩文章及资源，请关注我的微信公众号：『mculover666』。</font>**

![mark](http://mculover666.cn/image/20190811/gKNrs8CqezFQ.jpg?imageslim)

---
title: 搭建hexo博客
date: 2019-03-07 19:37:10
tags: 入门
---
平时查资料，看了那么多页面，发现好多博客底部都标识着“Hexo”，比如[网易考拉前端团队](http://blog.kaolafed.com/)。知道 Hexo 是一个博客框架。但一直没入坑。

我问自己，需要博客吗？日常有搭建自己的知识体系的习惯，但都写在印象笔记里，算上碎碎念和与业务相关的现也有累计百余篇，没有对外公开。之前在 [W3Cfuns（现更名为：QDFuns）](http://www.w3cfuns.com/) 写过几篇，但没坚持下去。一是花时间，二是我实在认为（也是事实），网上太多人比我写得好，我再写有什么意义呢？
<!-- more -->

直到这两天在掘金看到一篇文章，大意说的是作者一开始也是和我这样想，但是自从尝试了向外总结，发现收获远远大于过往。我忘了他怎么说的了，总之：
![](/images/sticker-heart.jpg) 

怎么说呢，非常赞同在微博上看到的这句话：对表达怀有羞耻感，是成长最大的敌人。这也是我的不足之处。

第一篇总结送给 [Hexo](https://hexo.io/zh-cn/) 。

## 前提
在电脑中已经安装好 Node.js 和 Git。此部分不作过多介绍。

## 安装 Hexo
安装好 Nodejs，使用 npm 开始安装 Hexo。
```
$ npm install -g hexo-cli
```

## 建站
先创建一个文件夹，或等会使用命令创建文件夹都行。
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

运行程序，在浏览器中访问本地 localhost:4000 即可看到博客已经搭建成功。
```
$ hexo server
```


## 关联Github
### 1. Github 创建一个新的 repositories。
每个账号只能有一个仓库来放个人主页，而且仓库的名称必须是：Github账号.github.io，这是特殊的命名规定。
![](https://i.imgur.com/G91wiJ6.png)

部署到 Github 的原理
（1）Github账号.github.io 这个仓库的 master 里的 html 文件可以直接通过网址 Github账号.github.io 访问。
（2）`hexo -g` 会生成一个静态网站，可以直接访问。

### 2. 配置`/_config.yml`
搜索找到相应配置进行修改。
```
url: http://xxx.github.io

deploy:
  type: git
  repository: https://github.com/xxx/xxx.github.io.git
  branch: master
```


## 配置
核心文件：`/_config.yml`
每篇文章的开头内容支持设置网站的名称、描述、作者名称和主题的选择等。
```
title: xxx
date: xxx
tags: xxx
```


## 主题
Hexo 默认的主题不太好看。没有好看的主题，就像去健身不先购入一套装备，缺乏动力。因此，写作之前，必须先快速选个主题。
网上热门的主题说来说去就是以下几个。

- [Tranquilpeak](https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak)
- [NexT](https://github.com/iissnan/hexo-theme-next)
- [Yilia](https://github.com/litten/hexo-theme-yilia)
- [cafe](https://github.com/giscafer/hexo-theme-cafe)（阮一峰大大同款）
- ……

主题的选择主要考虑两点：一是符合自己的审美，二是项目维护度高，随之扩展性也强。
综上所述，不免其俗，我和最广大的网友们选择一致，用NexT。这个主题做得确实好。甚至它还有自己的[官网](https://theme-next.iissnan.com/)。
这个主题提供了非常多的功能，有兴趣对博客功能进行扩展的欢迎到官网文档查看，文档写得很详细。

## 写作
### 1. 新建一篇文章
``` bash
$ hexo new "文章标题"
```

### 2. 修改文章信息

### 3. 编辑文章
使用 Markdown 语法开始写文章。

### 4. 推送
``` bash
$ hexo clean 	// 删除本地静态文件（Public目录）
$ hexo generate  //生成本地静态文件（Public目录）
$ hexo deploy    // 将本地静态文件推送至github（hexo d）
```
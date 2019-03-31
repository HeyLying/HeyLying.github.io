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

## 特殊场景
### 我的需求
我的 hexo 项目是在公司电脑里搭建好的，写完文章将静态资源代码推送至 GitHub 中（有仅且一个 master 分支），现在我有这样一个需求：我回到家后，也想抽空写文章更新到线上。或者说以后我还有第三台电脑里也想这么操作——这怎么处理合适呢？

### 解决思路
观察发现，hexo 上传到 Github 上内容的其实只有生成的 public 文件夹，其他源码部分没上传。
那么我们主要就是管理好源码部分，把它上传上去就解决问题了。这样以后想在哪写作、部署和推送就执行一句 git clone 即可。
当然，其实把它放在网盘等等之类管理同理，也是可行的。
或者，采用究极简单的办法：仅异地写文档，将文档上传到网盘等，最后所有文章仍在公司电脑里推送——当新电脑不方便安装 Node.js 和 Git 等搭建环境时非常适用。

存放在 Github 中管理有两种方法：
- 新建一个项目，存放源码。
- 在 hexo 原项目中新建一个分支，存放源码。
即 master 存放生成后的内容，new branch 存放生产前的内容。

由于它俩本质是同一个项目，所以我采用了第二种方法。

### 操作流程
- 先在 Github 中对本项目新建一个分支，假设命名为 source，并设置为默认分支。
此时 source 的内容与 master 一致。
- 在公司电脑里执行 git clone，将 source 克隆到本地。
- 删除 source 里的内容， 将原项目完整复制放进来，也就是指公司电脑中的 hexo 源项目，此部分有两块内容需要剔除。
1.务必除去 node_modules（内容太大），建议删去 .deploy_git 和 public（两者会重新生成） 文件夹。
2.务必除去 themes 中的主题文件夹内的 .git 文件夹（如果有），否则到时 git push 到分支的 themes 文件夹为空。
之前我没删除这个 .git 就推送上去，结果远端的主题内容为空，于是我在本地重新下载一份主题，然后启动服务器，出现白屏或乱码各种情况……博客无法正常显示。删掉重新再推送，远端的 themes 文件夹有内容了，克隆到本地的博客显示也正常了。

处理好拷贝，确认无误后，将其推送到远端的分支。
```
git add .
git commit –m "add branch"
git push 
```

刷新Github，现在分支的内容为：
![](/images/hexobranch.png) 

ok，然后现在回到家中，将分支克隆到本地。
```
npm install hexo-cli -g

cd xxx.github.io
// 尤其确保：
// npm install hexo-server --save
// npm install hexo-deployer-git --save
npm install

```

现在，hexo 源项目即可在家中电脑运行起来了（localhost:4000），可以进行如上常规步骤，写作、生成和部署。

### TIPS
在 /_config.yml 中 hexo 部署生成后文件指定master分支。
```
deploy:
  type: git
  repository: https://github.com/xxx/xxx.github.io.git
  branch: master  // 指定master
```

每次写作前，记得与远端同步。
```
git pull
```

每次写完后，记得上传源文件到远端。
```
git add .
git commit –m "xxxx"
git push 
```

知乎同款问题：https://www.zhihu.com/question/21193762
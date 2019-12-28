---
title: 如何为VS Code开发一个插件
date: 2019-12-15 22:55:21
categories: 工具
---

VS Code是一个出色的代码编辑器，但真正使它强大的是它可用的扩展（也称为插件）。

首先要说明一下，可能有些人喜欢用 Webstorm/Sublime等其他编辑器或IDE，甚至我之前写PHP业务，用过一阵Phpstorm——都没关系，没用过VS Code的人也肯定知道它。认识下VS Code插件，知道VS Code的插件怎么开发，依旧可以当作扩展自己技术见识。

VS Code作为一个轻量级代码编辑器，和Sublime类似，本身功能并不全面，靠安装外部插件来增强开发体验。

于是趁项目组轮到我开展一次分享会，我特此花了两个周末，参考网上教程，将VS Code的插件做了一番学习。

<!-- more -->

我主要是阅读以下博客内容进行学习。这是我找到的为数不多对VS Code插件开发较为全面而通俗的讲解，特此感谢作者！

https://www.cnblogs.com/liuxianan/p/vscode-plugin-overview.html



## VS Code插件开发

### 前言 有趣的插件

插件可以做的事情还挺有意思的。首先介绍两个插件：

坤坤鼓励师 提醒休息闹钟
https://marketplace.visualstudio.com/items?itemName=sakura1357.cxk

Thief-Book 摸鱼看书神器
https://marketplace.visualstudio.com/items?itemName=C-TEAM.thief-book

加上平时自己用的插件，给我的感受就是，vscode的插件能做的事可以有很多。



### 前言 插件能做什么

- 不受限的本地磁盘访问
- 自定义命令、快捷键、菜单
- 自定义跳转、自动补全、悬浮提示
- 自定义设置、自定义欢迎页
- 自定义Webview
- 自定义左侧功能面板
- 自定义颜色、图标主题
- 新增语言支持
- Markdown增强
- 其他（状态栏修改、通知提示……）

这里有一点需要注意，开发出来的插件没法对VS Ccode本身界面作大篇幅的改动，只能顶部加一点，或侧边加一点。目的是引导扩展开发者保持UI的简洁和视觉风格的统一。毕竟用户的审美千种万种，太不可靠。



### 学习目录

![](/images/7.png) 



### 一、Hello World

```
npm install -g yo generator-code
yo code
```

运行yo命令，yeoman就会生成generator-code脚手架里配置的内容，它配了什么呢，里边是一个插件规范的目录结构，并且已经实现弹一个helloworld弹窗的功能。

>  yo来自脚手架Yeoman，一句话概括它的主要功能：帮我们快速建立一个项目的基础目录结构和构建任务。

按下Ctrl+Shift+P，输入HelloWorld运行插件。
https://code.visualstudio.com/api/get-started/your-first-extension

1. 项目结构
package.json：清单文件
extension.js：插件入口文件
2. 代码解读
    main：主入口
    contributes.commands：注册了extension.sayHello的命令。具体由src/extension.js实现。
    activationEvents：插件的激活事件，告知VS Code什么时候执行该命令。
3. 运行调试
    .vscode/launch.json
    弹出新窗口，会注明扩展开发宿主。新窗口已经加载了新写的插件。

4. 其他：重新加载、开发语言



### 二、package.json

1. 常用配置
https://code.visualstudio.com/api/references/extension-manifest
2. activationEvents 配置插件的激活事件
[示例] onLanguage:javascript：只要打开了js文件，插件就会被激活。
[示例] 配置为*：只要一启动VS Ccode，插件就会被自动激活。官方不推荐。
https://code.visualstudio.com/api/references/activation-events
3. contributes 扩展VS Code的功能
https://code.visualstudio.com/api/references/contribution-points



### 三、命令

1. vscode.commands.registerCommand 注册普通命令
2. vscode.commands.registerTextEditorCommand 注册文本编辑器命
3. vscode.commands.excuteCommand 复杂命令
   https://code.visualstudio.com/api/references/commands
4. vscode.commands.getCommands 获取所有命令
   https://code.visualstudio.com/api/references/vscode-api



### 四、菜单

```javascript
// package.json 
"menus": {
      "editor/context": [
        {
          "when": "editorFocus && resourceLangId == javascript",
          "command": "extension.menuShow",
          "group": "z_commands"
        }
      ],
      //...
 }
```



#### 1. 菜单的配置

一个菜单的添加配置需要以下这些内容的设置。

editor/title：定义菜单出现在哪里
when：定义菜单什么时候出现
command：定义菜单被点击后执行的操作
alt：定义备用命令，按住alt键打开菜单时将执行对应命令
group：定义菜单的分组



#### 2. 菜单出现在哪里

文件查看器右键菜单 explorer/context

编辑器
  右键菜单 editor/context（在编辑器中）
  标题栏菜单 editor/title
  标题栏右键菜单 editor/title/context

左侧视图
  文件查看器分栏 view/title
  调试视图分栏 view/item/context
搜索框下方菜单 commandPalette

SCM（源码管理）视图
  标题栏菜单 scm/title
  文件分组菜单 scm/resourceGroup/context
  文件状态菜单 scm/resource/context
  文件变动菜单 scm/change/title

调试视图
  调用栈右键菜单 debug/callstack/context



#### 3. 菜单什么时候出现

resourceLangId == javascript：当编辑的文件是js文件时
resourceFilename == test.js：当当前打开文件名是test.js时
isLinux、isMac、isWindows：判断当前操作系统
editorFocus：编辑器具有焦点时
editorHasSelection：编辑器中有文本被选中时
view == someViewId：当当前视图ID等于someViewId时
……
多个条件可以通过与或非进行组合

https://code.visualstudio.com/docs/getstarted/keybindings#_when-clause-contexts



#### 4. 菜单的分组

不同的菜单拥有不同的默认分组。

（1）组间排序
editor/context中有这些默认组
navigation：永远排在最前面
1_modification：更改组
9_cutcopypaste：编辑组
z_commands ：最后一个默认组，其中包含用于打开命令选项板的条目

（2）组内排序
默认同一个组的顺序取决于菜单名称，如果想自定义排序的话可以再通过@<number>的方式来自定义顺序。
"group": "navigation@2"   //强制放在navigation组的第2个
https://code.visualstudio.com/api/references/contribution-points#contributes.menus



### 五、快捷键

设置的快捷键会出现在VS Code的首选项-键盘快捷键设置界面。

```javascript
"keybindings": [
      {
        "command": "extension.sayHello",
        "key": "ctrl+f",
        "mac": "cmd+f10",
        "when": "editorTextFocus"
      },
      //...
]
```



### 六、语言

语言类的API：

1. vscode.languages.registerDefinitionProvider 跳转到定义
2. vscode.languages.registerHoverProvider 悬停提示
3. vscode.languages.registerCompletionItemProvider 自动补全
    参数：触发提示的字符列表



### 七、Webview

1. 定义和Demo
    vscode.window.createWebviewPanel

2. 什么时候用Webview
    官方建议谨慎使用。因为Web视图占用大量资源，并且设计不佳的网页容易让人感到诡异，些许不适。
    
3. 开发
    （1）加载本地资源
    出于安全考虑，Webview默认无法直接访问本地资源。要使用vscode-resource:协议（类似file:）。
    （2）主题适配 vscode-light / vscode-dark / vscode-high-contrast
    （3）消息通信 
    acquireVsCodeApi()返回极简版vscode对象。
    
    https://code.visualstudio.com/docs/extensions/webview



### 八、代码段

新增一个代码段需要设置以下内容。

for循环：代码片段的名称
prefix：输入什么单词会触发代码片段
body：一个数组，存放代码片段的内容，每一行一个字符串
description：代码片段的描述
${1: xxx}：占位符，数字表示光标聚焦的顺序。1表示默认光标落在这里，按下回车或者tab跳到2的位置，以此类推。xxx表示此位置的默认值，可省略，比如直接写$5。

```javascript
"snippets": [
      {
        "language": "javascript",
        "path": "./snippets/javascript.json"
      }
]
```

```javascript
//./snippets/javascript.json
{
    "for循环": {
        "prefix": "for",
        "body": [
          "for (const ${2:item} of ${1:array}) {",
          "\t$3",
          "}"
        ],
        "description": "for循环"
    }
}
```



### 九、设置

每一个插件可以创建属于自己的专属设置项，这个设置项会出现在 首选项-设置-扩展 中。

```
// 如果没有设置，返回undefined
const result = vscode.workspace.getConfiguration().get('vscodePluginDemo.yourName ');

// 最后一个参数，为true时表示写入全局配置，为false或不传时则只写入工作区配置
vscode.workspace.getConfiguration().update('vscodePluginDemo.yourName', ‘lying', true);
```



### 十、调试

1. 原窗口的调试控制台
    插件的console.log()也会在这里显示。调试控制台只打印常规日志，语法错误并不会显示在这里。

2.  扩展窗口的调试控制台
    插件相关信息不多。

3. 原窗口的开发者控制台
    插件相关信息不多。打开方式：帮助 → 切换开发人员工具

4. 扩展窗口的开发者控制台【注意】
    插件的所有信息（包括报错）在这里显示。打开方式：帮助 → 切换开发人员工具

5. Webview控制台
    使用扩展窗口的开发者控制台，只能查看到<webview></webview>标签。
    正确的打开方式：Ctrl+Shift+P → 打开Webview开发工具

6. 具体调试
    在需要调试的地方打个断点，然后按F5执行。

7. VS Code的源码
    所有的VS Code的api都可以在index.d.ts找到。

8. 查看插件存放目录

| os      | path                                                 |
| ------- | ---------------------------------------------------- |
| windows | %USERPROFILE%\.vscode\extensions（cd %USERPROFILE%） |
| macOS   | ~/.vscode/extensions                                 |
| Linux   | ~/.vscode/extensions                                 |

9. 插件为什么没生效
    （1）activationEvents是否已添加？开发的时候可以设置为：*
    （2）代码是否报错了？插件的错误需要打开控制台查看。



### 十一、发布

1. 发布方法
   方法一：把文件夹发给别人，将其放进VS Code的插件存放目录，然后重启。一般不推荐。
   方法二：打包成vsix，然后发送给别人安装（扩展-右上角-从 VSIX 安装）。
   如果插件涉及机密不方便发布到应用市场，可以采用这种方式。
   方法三：注册开发者账号，发布到官网应用市场，无需审核。

2. vsce 打包工具
   方法二、三都需要借助vsce这个工具。

   ```
   npm i vsce -g
   vsce package
   ```

3. 发布到应用市场
   （1）注册microsoft账号 
   https://login.live.com/
   （2）创建token 
   https://aka.ms/SignupAzureDevOps
   （3）创建发布者账号
   vsce create-publisher your-publisher-name
   会依次要求输入昵称、邮箱和token等。
   （4）发布
   其中README.md文件默认会显示在插件主页。

   ```
   vsce publish
   ```

   发布成功后大概需要过几分钟才能在应用市场搜到。



## VS Code插件市场

VS Code有太多优秀的插件~

- View In Browser

- vscode-icons

- Prettier

- carbon-now-sh

- Turbo Console Log

- GitLens

- Local History

  https://blog.csdn.net/qq_41139830/article/details/85221330
  https://marketplace.visualstudio.com/VSCode

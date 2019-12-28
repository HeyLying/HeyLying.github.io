---
title: 梳理ESLint和Prettier的使用
date: 2019-12-28 21:13:17
categories: 工具
---

对于ESLint和Prettier，之前知道是知道，但一直没理清之间的关系，总是有些模糊——模糊倒也是对的，因为它俩确实有些交集。

前阵子VS Code的Prettier插件升级，导致我的VS Code里的setting.json相关语法提示deprecated。没办法，忍受不了编辑器里的波浪线，我只能硬着头皮去解决。

我的问题：去掉了这个语法，现在在项目里，要如何怎么配合使用ESLint和Prettier？

<!-- more -->



**Linter和Prettier的区别**

ESLint、TSLint等统称为Linter。所以首先，我们先搞清楚Linter和Prettier的区别。

在格式化代码方面， Prettier确实和ESLint等有重叠，但它们的侧重点不同。

首先，梳理一遍质量和格式的区别。

- 代码质量问题指的是：未使用变量、三等号、全局变量声明等问题。
- 代码格式问题指的是：单行代码长度、tab长度、空格、逗号表达式等问题。

ESLint侧重代码质量，Prettier侧重代码格式。它们的具体区别如下。

- ESLint的主要工作是检查代码质量并给出提示，它可以检测出项目代码中潜在的问题，比如使用了某个变量却忘记了定义。它也提供一些格式化功能，但涉及范围很有限。
- Prettier全身心投入在格式化代码方面，格式化功能强大得多，能够统一你或者你的团队的代码风格，但不能提示变量未定义之类。

参考资料：https://github.com/prettier/prettier-ESLint/issues/101#issuecomment-313233479



**Linter和Prettier配合使用**

为什么要两者配合使用？

如果觉得ESLint提供的格式化代码够用了，也可以不使用Prettier。但总体来说，Prettier的格式化还是覆盖更加全面。

ESLint和Prettier相互合作的时候会有一些问题，对于他们交集的格式化部分规则，ESLint和Prettier格式化后的代码格式不一致。导致的问题是：当你用Prettier格式化代码后再用ESLint去检测，会出现一些因为格式化导致的 warning。这个时候有两个解决方案：

- 运行Prettier之后，再使用ESLint --fix格式化一把，这样把冲突的部分以ESLint的格式为标准覆盖掉，剩下的 warning就都是代码质量问题了。
- 在配置ESLint的校验规则时候把和Prettier冲突的规则disable掉，然后再使用Prettier的规则作为校验规则。那么使用Prettier格式化后，使用ESLint校验就不会出现对前者的warning。

专业的问题交给专业的人做，参考官方文档和网上教程，我选择了后者。主要的设置步骤如下。

- 关闭ESLint的样式部分的规则。
- 将Prettier集成到ESLint的工作流中。



Prettier提供了一些与ESLint配合使用的工具或插件，有好几个，查了资料才搞清其中区别。

参考网上资料：

![](/images/10.png)



**实现上述步骤的具体方案：使用eslint-config-prettier 和eslint-plugin-prettier**

```
npm i -D prettier eslint-plugin-prettier eslint-config-prettier
```

```
// .eslintrc.js
module.exports = {
  extends: [
    // 各种你需要继承的配置列在前面
    'airbnb'
    'plugin:vue/recommended',
    // prettier 规则列在最后
    'plugin:prettier/recommended',
    'prettier/vue',
  ],
}
```

参考资料：

https://www.meteorlxy.cn/posts/2019/08/05/understand-and-use-prettier.html

https://stackoverflow.com/questions/44690308/whats-the-difference-between-prettier-ESLint-ESLint-plugin-prettier-and-ESLint



**附录**

**ESLint airbnb-base和standard风格的简单对比区别**

1. airbnb-base
   - 每个句末带分号
   - function后不空格
   - 强制换行符

2. standard
   - 最后一个元素句末不带分号
   - function后有空格
   - 无强制换行符
---
title: 为何我最终放弃升级webpack4
date: 2018-04-19 09:19:01
tags: [code]
cover: /images/webpack4/webpack.png
description: 升级webpack4的种种艰辛
---

# 为何我最终选择放弃升级webpack4
webpack是当前最火的一款前端工具，它能处理包括CommonJS，AMD，ESM等各种模块与包括CSS，图片，文件等几乎各种前端所需的资源，并将其打包为各种资源模块。虽然webpack的本质是模块分析加载工具，但是丰富的插件系统也赋予了其前端构建的能力，可以替代grunt/gulp，提供诸如代码压缩混淆，图片转base64，具备热更新功能的开发服务器等功能。Light团队内的前端工程也或多或少地采用了webpack的技术，但在我们众多的前端项目还在使用webpack2时，webpack却已经迎来了v4版本，我们跳过v3，来看看最新的webpack有哪些重大变化与功能特性。

## 重大变化
- 不再支持Node.js 4。
- webpack配置文件中新增`mode`配置项，开发者可以选择使用`none`,  `development`或`production`，默认为`production`。在`development`模式下，会有更加完善的报错信息，提供调试工具及更快的增量编译速度。`production`模式下，开启了各类代码打包优化与模块作用域提升，使得bundle尺寸更小，运行更快。看到这，我流出了激动的泪水，我们再也不需要通过判断`process.env.NODE_ENV`来确定运行环境并决定是否添加各类插件了，如今只要简单地设置`mode`值，遵循webpack默认设置，就能轻松上手了。
- `import`总是返回一个对象，`CommonJS`模块会封装在`default`属性中。所以引用`CommonJS`的代码将修改为`import('xxx').default`。
- `NoEmitOnErrorsPlugin `，`ModuleConcatenationPlugin `，`NamedModulesPlugin `，`CommonsChunkPlugin `等插件已经默认集成在`devlopment`或`production`环境中，不再需要单独配置。
- 支持直接`import`JSON，在不需要`json-loader`。

## 性能升级？
在冗长的功能特性列表中，我的目光聚焦在了`Performance`这一项上，webpack4为我们带来了更多性能上的提升，包括了压缩代码时默认开启优化策略，编译时多处性能提升，尤其是增量编译速度将更快。看到这，我已经跃跃欲试了，毕竟在日常的开发过程中，我们已经感受到了日渐庞大的前端工程对webpack性能造成的压力，很多同事可能都养成了先输入`npm run watch`，泡一杯杯咖啡再回来继续开发的习惯了。既然如此，我们心动不如行动，马上来测试下webpack4的速度如何。

## 工程简介
我这里选择 [lighting UI](https://github.com/HS-Light/weex-ui)工程来进行webpack升级测试。`lighting UI`工程是一个基于`weex-ui`不断拓展完善的UI库工程，包含了约60个左右的UI组件源码(.vue文件)及示例页面。
![Alt text][1]


![Alt text][2]

在工程的webpack配置文件中，导出了两个webpack配置，其中一个通过`vue-loader`与`babel-loader`生成web环境中使用的js文件，另一个配置通过`weex-vue-loader`与`babel-loader`生成在native环境中使用的js文件。


## 版本升级
接下来我们修改`package.json`中的webpack版本，从`^2.7.0`变为`^4.5.0`，执行`npm install`重新安装依赖后再执行`npm run build`。
- 首先跳出了安装`webpack-cli`的提示，webpack4中webpack命令行被分离到独立的一个npm包中，我们输入`yes`继续。
- 第一次编译失败，错误来自于`CopyWebpackPlugin`，我们首先想到的是升级插件版本，将插件升级到最新版本后重新编译，我们又得到了来自`vue-loader`与`weex-vue-loader`的报错，同样，我们也将`vue-loader`从`^12.2.2`升到最新版本`14.2.2`。再次尝试，来自`vue-loader`的错误也消失了。
- 然而将`weex-vue-loader`更新到最新版本后，编译时错误也依然存在，显然`weex-vue-loader`更新过慢，已经不兼容webpack4，这个时候升级之路似乎陷入了僵局。

## 兼容错误
![Alt text][3]


![Alt text][4]

不得已，我们只能根据编译时错误来排查`weex-vue-loader`源码中不兼容的代码。
根据错误，我们能够简单地得出结论，在loader的上下文中已经不存在`options`这个属性，所以才出现了当前错误，同时我们也在webpack4的`Removed features`中得到了验证。
![Alt text][5]

那么，我们如何去拿到这个神秘的`options`呢，通过输出loader的上下文，我们发现在`this._compiler`中存在一个`options`属性，我们大胆猜测`this._compiler.options`可以替代原有的`options`，我们的猜测看起来没有问题，在修改了相关代码之后，编译终于成功了。这个时候可以完成我们最初的心愿——测试webpack4编译速度。

## 对比结果
在平时工作过程中，我们更加关注的是在工程在开发模式下的编译速度，于是我们先修改原有的webpack配置文件，将`devtool`的值从`source-map`设置为`eval`，这样我们能以较快的速度得到`map`文件而不影响调试。而在webpack4的配置文件中，我们同样修改`devtool`的值为`eval`，并加上`mode`值为`development`的配置项。
一轮测试下来，结果却让人大跌眼睛，单独编译`[name].web.js`时，两者时间相差无几；而单独编译`[name].native.js`时，webpack4的时间远远大于webpack2的时间，同时也出现了webpack2编译时不存在的语法警告。到这里，我似乎恍然大悟，正如在升级公告中写到的那样，webpack4在开发阶段默认开启了语法检查与`devtool`甚至更多我们无法配置的优化项，一切只为了在更好的开发体验与更快的编译速度之间找到平衡点。

## 总结
折腾到这，我们的这次测试也宣告失败，但这也并非是毫无意义的一次探索。
我们在升级webpack4时体会到了`mode`配置项的引入对初始化工程配置带来了极大的便利，虽然遇到了`weex-vue-loader`不兼容的问题，使得该工程无法升至webpack4，但在其他工程中我们可以有取舍地去选择是否升级webpack4，毕竟也有相当多的`loader`与`plugin`已经支持最新的webpack了。最后希望webpack4的生态尽快完善，给开发者带来更好的体验！

[1]: /images/webpack4/baseconfig.png
[2]: /images/webpack4/exportconfig.png
[3]: /images/webpack4/error.png
[4]: /images/webpack4/source.png
[5]: /images/webpack4/fetures.png
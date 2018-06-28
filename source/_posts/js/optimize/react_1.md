---
title: 如果不能忍受，何不试着改变--前端项目优化心得分享
date: 2018-04-10 14:46:25
categories: 编程
tags: [JS,优化,前端]
---

从去年年底开始，重新接触这个项目，在开发的过程中，无比得痛苦，

最后实在是不能忍，所以就打算做些改变。

<!--more-->
一、不能忍受什么？
<div style="font-size:28px">
<div style="color:#F06292">慢---编译速度慢，生产、开发过程中都浪费很多时间，效率低下</div>
<div style="color:#2c3f51; font-size:20px;">一次本地启动要2分多种，修改一次代码要20多秒，这实在是让人很痛苦</div>
<div style="color:#039BE5">乱---代码风格混乱，每个人写得代码格式差别很大，阅读代码很吃力</div>
<div  style="color:#2c3f51; font-size:20px">修改别人代码的时候，得花点时间才能看懂代码，如果要修bug就更得多花时间，这也很痛苦</div>
 <h3 style="color:#F06292">二、慢</h3>
 
#### 慢的原因是什么？
我们的代码是`react`全家桶，打包工具用了`gulp` +` webpack`组合方式，慢主要就是慢在了打包

`gulp`和`webpack`的分工：

 `webpack`负责将项目中用到的`js、scss/css img、png`等模块化打包

`gulp`负责`css、js`代码检查，并控制整个打包的流程，指导`webpack`打包
#### 发现的问题主要有：


1、所用的库文件版本太旧： `webpack`  *v1.x*

2、没有进行按需加载（只加载用到的组件）、把整个库都打包进来，导致速度慢、包很大

3、库文件可缓存，但没有单独打包,

4、`normalize.css` 重复加载



#### 采取的措施有：

1、`webpack` 从v1.x 升级到v3.x
- 一次破坏性升级

2、移除`gulp`,`webpack` *v3.x*已经完全可以独立完成整个打包流程
- 代码检查： `webpack`可以做
- 流程控制：PC、H5、Server：`webpack` 可以做到并行打包
- 移动静态文件：直接用`nodejs`实现
- version:` linux`命令实现

到此，所有`gulp`的任务都被替换，没有理由不抛弃。

3、修改代码，`lodash ` 、`antd`以按需加载的方式，正确引入包


3、使用`cdn`，将库文件的`css`移除，以`cdn`的方式引入

- `antd` 和我们系统本身都依赖于`normalize.css`，所以使用cdn将`normalize`和`antd`的css分别引入[链接](http://www.bootcdn.cn/normalize/)

4、移除不需要的，比如	`moment`国际化的代码
>上下文替换插件(ContextReplacementPlugin)
>上下文(Context) 与一个带表达式的 require 语句 相关，例如 require('./locale/' + name + '.json')。遇见此类表达式时，webpack 查找目录 ('./locale/') 下符合正则表达式 (/^.*\.json$/)的文件

```
  new webpack.ContextReplacementPlugin(
    /moment[\\/]locale$/,
    /^\.\/(zh-cn)$/
  ),
```


5、抽取`css`代码只在生产模式做--区分生产和开发模式

- 生产模式要把所有的css抽离为一个文件，防止页面闪动，但这样比较费时间，开发过程中根本没有必要

6、`webpack`中 `hash` 、`chunkhash` 、`contenthash` 代码缓存
- hash和整个项目相关
- chunkhash 和整个chunk的内容有关
- contenthash 抽取css文件的插件中提供的

8、用 `echart`替换老旧的`d3`

-	`d3`比较老旧，功能比较单一，文档又很烂
-  `echart`功能强大，还可以按需加载，文档清晰，例子很多

8、关于`react` 	`react-router` 的升级，
 - `react` 继续用的v15,
 - `react-router` 从v2升级到v3, 暂时不升级到v4

-----------

#### 升级过程中使用的工具

##### A. `webpack`  官方提供的在线分析工具 [地址](http://webpack.github.io/analyse/)


-------------


##### B. 本地分析工具

#### 效果：

| -          |    优化前 | 优化后  |
| :--------: |:--------:| :--:   |
| 生产模式    | 5min+    | 3min+- |
| 开发模式    | 2min+   | 1min-  |
| 热更新      | 20s+     | 5-7s   |

----------------


----------

 <h3 style="color:#039BE5">三、乱</h3>

####  乱的原因是什么？

- 项目从开始到现在，有很多人参与并贡献过代码，每个人技术层次不齐，编码习惯各不相同，缺少一种规范，帮助大家养成良好的编码习惯，可以降低错误发生的几率，也比较容易发现错误。
- 以前代码检查只在生产模式做，导致每次开发完后还得在用生产模式运行一边，而且检查的内容很简单: `===`

#### 解决方法：

使用代码检查工具：

 `Eslint`

 `Airbnb` [JavaScript Style Guide](https://github.com/airbnb/javascript)

`stylelint`  


我们现在的代码存在很多问题，不得不把很多功能关掉。



#### 效果：

有eslint帮我们检查代码，可以减少很多不必要的错误，如果开发过程中代码出现问题，会直接出现在屏幕上.

大家都能写出一致风格的代码，降低阅读代码和修改代码的成本




使用`husky`；`git commit` 之前检查代码，如果失败，不能`commit`

---------

### 四、其他

1、自动化测试: `jest`：

2、编辑器 `vscode` ：超级好用，从来没崩溃过。	`webstrom `占用内存

------

### 总结

因为自己忍受不了开发过程中的慢和乱，所以就想着做点优化，提高自己的工作效率。

然后想到大家在开发过程中可能也会遇到类似的问题，并且也想着怎么提高自己的工作效率。

那这次的分享主要就是给大家一个范例，希望能抛砖引玉，希望对大家在工作中有一些启发，能能好更高效的完成自己的工作。


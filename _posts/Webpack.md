---
layout: post
title: 面试-Webpack
date: 2025-08-14
categories:
author: Cookinne
cover: /assets/img/posts/webpack.jpg
tag: 面试
---

## entry output

entry: 入口文件， 可以是数组或者对象形式，可以配置多入口

```js
entry: 'index.js'

entry: ['index.js']

entry : { main: 'index.js' }

// 多页面，多入口
entry: {
    a: a.js,
    b: b.js
}
```

output: 输出文件，配置路径，文件名，chunk 文件名等。

```js
output: {
    path: xxx绝对路径,
    filename: [name].[id].[hash].[chunkhash].[contenthash].js
    chunkFilename: [name].[id].js, // 这个name并不是ntry入口的名字， 而是魔法注释 /* webpackChunkName: 'myChunk' */
    publicPath: xxx, // 指定app运行在哪个路径， 如根路径 “ / ” 或者子路径 “ /subpath/ ”
}
```

## loader 和 plugins 的区别

loader: webpack 自身只支持 js 和 json 这两种格式的文件，对于其他文件需要通过 loader 将其转换为 commonJS 规范的文件后，webpack 才能解析到。Loader 是在打包构建过程中用来处理源文件的（JSX，Scss，Less..），一次处理一个。

plugin: 插件用于扩展 webpack 的功能，作用于 webpack 的整个生命周期，比 loader 更强大。plugin 可以监听 webpack 的生命周期事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

## 有哪些常见的 Loader？

```js
css - loader,
  style - loader,
  sass - loader,
  less - loader,
  url - loader,
  file - loader,
  babel - loader,
  eslint - loader;
```

## 有哪些常见的 Plugin？

- HtmlWebpackPlugin： 生成 HTML 文件，并将打包的 bundle 和 chunk 引入到 HTML 中
- webpack.DefinePlugin：在 js 代码中使用自定义变量
- InterpolateHtmlPlugin：在 index.html 中使用自定义变量
- MiniCssExtractPlugin: 该插件将 CSS 提取到单独的文件中，可以指定输出文件名
- CleanWebpackPlugin： 清空上次打包的文件夹
- CopyPlugin： Copy 指定文件夹下的内容到指定文件夹
- SplitChunksPlugin： 用来提取第三方库和公共模块
- TerserPlugin： 压缩 js 代码, 一般用于 optimization 选项
- OptimizeCSSAssetsPlugin： 压缩 css 代码，一般用于 optimization 选项

## webpack 的构建流程是什么?

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
- 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
- 确定入口：根据配置中的 entry 找出所有的入口文件；
- 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
- 完成模块编译：在经过第 4 步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
- 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
- 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

## 分别介绍 module, chunk, bundle 是什么

- module: 简单来说，你通过 import 语句引入的代码块，都叫 module。css，图片等也都叫 module
- chunk: 是 webpack 根据功能拆分出来的，包含三种情况：
  - 项目入口（entry）chunk，也就是 main chunk，一般这个 chunk 包含了你大部分的代码。
  - 通过 import()动态引入的模块，会被打包为子 chunk
  - 通过 splitChunks 拆分出来的代码, 也是子 chunk
- bundle: 是 webpack 打包之后的各个文件，一般就是和 chunk 是一对一的关系。bundle 就是对 chunk 进行编译压缩打包等处理之后的产出。

## 如何提高 webpack 构建速度

1. 使用 cache-loader 缓存结果
2. 使用 DLLPlugin 和 DLLReferencePlugin

DLLPlugin 这个插件是在一个额外独立的 webpack 设置中创建一个只有 dll 的 bundle，也就是说我们在项目根目录下除了有 webpack.config.js，还会新建一个 webpack.dll.config.js 文件。

webpack.dll.config.js 作用是把所有的第三方库依赖打包到一个 bundle 的 dll 文件里面，还会生成一个名为 manifest.json 文件。该 manifest.json 的作用是用来让 DllReferencePlugin 映射到相关的依赖上去的。

DllReferencePlugin 这个插件是在 webpack.config.js 中使用的，该插件的作用是把刚刚在 webpack.dll.config.js 中打包生成的 dll 文件引用到需要的预编译的依赖上来。就是说在 webpack.dll.config.js 中打包后比如会生成 vendor.dll.js 文件和 vendor-manifest.json 文件，vendor.dll.js 文件包含所有的第三方库文件，vendor-manifest.json 文件会包含所有库代码的一个索引，当在使用 webpack.config.js 文件打包 DllReferencePlugin 插件的时候，会使用该 DllReferencePlugin 插件读取 vendor-manifest.json 文件，看看是否有该第三方库。vendor-manifest.json 文件就是有一个第三方库的一个映射而已。

所以说 第一次使用 webpack.dll.config.js 文件会对第三方库打包，打包完成后就不会再打包它了，然后每次运行 webpack.config.js 文件的时候，都会打包项目中本身的文件代码，当需要使用第三方依赖的时候，会使用 DllReferencePlugin 插件去读取第三方依赖库。所以说它的打包速度会得到一个很大的提升。

## 如何用 webpack 来优化前端性能？

1. 压缩代码。js, css, image
2. 利用 CDN 加速， 所以文件需要有 hash 值
3. gzip
4. tree shaking
5. 提取公共代码
6. 代码分割， 避免太大

## tree shaking 原理

Tree shaking 是基于 ES6 模板语法（import 与 export），主要是借助 ES6 模块的静态编译思想，在编译时就能确定模块的依赖关系，以及输入和输出的变量

Tree shaking 无非就是做了两件事：

编译阶段利用 ES6 Module 判断哪些模块已经加载

判断那些模块和变量未被使用或者引用，进而删除对应代码

## webpack、rollup、parcel 优劣？

- webpack 适用于大型复杂的前端站点构建
- rollup 适用于基础库的打包，如 vue、react
- parcel 适用于简单的实验性项目，他可以满足低门槛的快速看到效果

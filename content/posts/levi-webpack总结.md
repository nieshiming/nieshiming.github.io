+++ 
draft = true
date = 2020-04-21
title = "levi-webpack总结"
description = ""
slug = ""
authors = []
tags = ["webpack", "八股文"]
categories = ["webpack"]
externalLink = ""
series = []
+++


<details>
  <summary>webpack 基本配置（可展开 && 收缩）</summary>

<pre><code>
const os = require('os')
const { resolve } = require('path')
const webpack = require('webpack')
const HappyPack = require('happypack')
const WebpackBar = require('webpackbar')
const CopyPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin')
const CircularDependencyPlugin = require('circular-dependency-plugin')

const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length })
const source = resolve(__dirname, '..', 'src')

module.exports = {
  context: source,
  entry: {
    app: './index.tsx',
  },
  watchOptions: {
    // 不监听的 node_modules 目录下的文件
    ignored: /node_modules/,
  },
  stats: {
    assets: true,
    builtAt: true,
    colors: true,
    chunks: false,
    children: false,
    env: true,
    entrypoints: false,
    errors: true,
    errorDetails: true,
    hash: true,
    modules: false,
    moduleTrace: true,
    performance: true,
    publicPath: true,
    timings: true,
    version: true,
    warnings: true,
  },
  resolve: {
    unsafeCache: true,
    mainFiles: ['index'],
    mainFields: ['main'],
    extensions: ['.ts', '.tsx', '.js'],
    modules: [source, 'node_modules'],
    alias: {
      '@': source,
      'react-dom': '@hot-loader/react-dom',
      moment: 'dayjs',
      '@components': resolve(__dirname, '../src/components'),
      '@ant-design/icons/lib/dist$': resolve(__dirname, '../src/assets/icons.ts'),
    },
  },
  module: {
    rules: [
      {
        test: /\.((ts|js)(x?))$/,
        exclude: /node_modules/,
        include: [source],
        use: 'happypack/loader?id=babel',
      },
      {
        test: /\.(jpe?g|png|gif|svg)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8 * 1024,
              outputPath: 'images',
              name: '[path][name].[ext]',
            },
          },
        ],
      },
    ],
  },
  plugins: [
    new WebpackBar(),
    new CopyPlugin([
      {
        from: resolve(__dirname, '../static'),
        ignore: ['dll/*'],
        to: resolve(__dirname, '../dist'),
      },
    ]),
    new HtmlWebpackPlugin({
      title: '新际通管理平台',
      filename: 'index.html',
      template: resolve(__dirname, '../public/index.html'),
      hash: true,
      minify: {
        removeRedundantAttributes: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true,
        removeComments: true,
        collapseBooleanAttributes: true,
      },
      // favicon: resolve(__dirname, '../public/favicon.ico'),
    }),
    new HappyPack({
      id: 'babel',
      threadPool: happyThreadPool,
      loaders: [
        'cache-loader',
        {
          loader: 'babel-loader',
          query: {
            cacheDirectory: './node_modules/webpack_cache/',
          },
        },
        'eslint-loader',
      ],
    }),
    new webpack.IgnorePlugin(/^\.\/locale$/, /dayjs$/),
    new HardSourceWebpackPlugin(),
    new CircularDependencyPlugin({
      exclude: /node_modules/,
      include: /src/,
      failOnError: true,
      allowAsyncCycles: false,
      cwd: process.cwd(),
    }),
  ],
}
</code></pre>
</details>

#### webpack 模块打包原理
- [webpack 模块打包原理](https://juejin.cn/post/6844903802382860296#heading-5)

#### webpack 热更新原理

##### 参考文章
- [Webpack HMR 原理解析](https://zhuanlan.zhihu.com/p/30669007)
- [轻松理解webpack热更新原理](https://juejin.cn/post/6844904008432222215)
- [120 行代码帮你了解 Webpack 下的 HMR 机制](https://juejin.cn/post/6973825927708934174#comment)

#### webpack 如何自定义loader，实现思路如何
> 带有副作用的内容转换器,函数, 从右到左执行，实质上是一个compose函数
> 将源文件经过转换输出新的结果，支持链式操作，其本质上就是一个函数

<!--more-->

```javascript

module.exports = function(source, sourceMap?, data?) {}

/** 同步loader */
module.exports = function (context) {
  const options = LeviLoaderUtils.getOptions(this);

  console.log('loader配置项:', options);

  const result = context.concat(`console.log("${options.message || '没有配置项'}");`);

  return result;
};


/** 异步loader */
// module.exports = function (context) {
//   let count = 1;
//   const options = LeviLoaderUtils.getOptions(this);
//   const callback = this.async();

//   console.log(options);
//   const timer = setInterval(() => {
//     count = count + 1;
//     console.log(count);
//   }, 1000);

//   setTimeout(() => {
//     callback(null, context);
//     clearInterval(timer);
//   }, 4000);
// };

    // webpack callnack完整签名如下
    this.callback(
        // 异常信息，Loader 正常运行时传递 null 值即可
        err: Error | null,
        // 转译结果
        content: string | Buffer,
        // 源码的 sourcemap 信息
        sourceMap?: SourceMap,
        // 任意需要在 Loader 间传递的值
        // 经常用来传递 ast 对象，避免重复解析
        data?: any
    );

```

参考文章: [Webpack 原理系列七：如何编写loader](https://zhuanlan.zhihu.com/p/375626250)
 
#### webpack 如何自定义plugin，实现思路是如何
> plugin本质上就是拥有apply放的对象，在webpack初始化解读会执行apply方法
> plugin 可以监听webpack各个生命周期广播出来的事件，在合适的实际，通过webpack释放的API来改变输出结果

```javascript
class Levi_Plugin {
  apply(compiler) {
    //不推荐使用，plugin函数被废弃了
    // compiler.plugin("compile", (compilation) => {
    //   console.log("compile");
    // });
    //注册完成的钩子
    compiler.hooks.done.tap("MyPlugin", (compilation) => {
      console.log("compilation done");
    });
  }
}

// 注册异步狗子
class Levi_Plugin {
  apply(compiler) {
    compiler.hooks.run.tapAsync("MyPlugin", (compilation, callback) => {
      setTimeout(()=>{
        console.log("compilation run");
        callback()
      }, 1000)
    });
    compiler.hooks.emit.tapPromise("MyPlugin", (compilation) => {
      return new Promise((resolve, reject) => {
        setTimeout(()=>{
          console.log("compilation emit");
          resolve();
        }, 1000)
      });
    });
  }
}

module.exports = Levi_Plugin;
```
- compiler对象包含了 Webpack 环境所有的的配置信息。这个对象在启动 webpack 时被一次性建立，并配置好所有可操作的设置，包括 options，loader 和 plugin。当在 webpack 环境中应用一个插件时，插件将收到此 compiler 对象的引用。可以使用它来访问 webpack 的主环境
- compilation对象包含了当前的模块资源、编译生成资源、变化的文件等。

##### 参考文档
- [compiler构造](https://www.webpackjs.com/api/compiler-hooks/)
- [手写一个webpack/plugin](https://zhuanlan.zhihu.com/p/40930680)
- [Webpack手写loader和plugin](https://juejin.cn/post/6888936770692448270#heading-8)

#### 如何题提升webpack构建速度
- 使用高版本的webpack和nodejs
- 启用多进程构建（happypack/thead-loader）
- 压缩代码
 - 使用mini-css-extract-plugin 提取公共css
 - terser-webpack-plugin 开启js多进程压缩
- 缩小打包作用域
 - exclude/include (确定 loader 规则范围)
 - resolve.modules 指定解析第三方包的目录位置 (减少不必要的查找)
 - resolve.extensions 尽可能减少后缀尝试的可能性
 - 合理使用alias,简化导入

- 提取页面公共资源
 - 使用 SplitChunksPlugin 进行(公共脚本、基础包、页面公共文件)分离(Webpack4内置) ，替代了 CommonsChunkPlugin 插件
 - 提取公共js资源 => splitChunks
    ```javascrit
    module.exports = {
        optimization: {
            splitChunks: {
                cacheGroups: {
                    utils: {
                        chunks: 'initial',
                        minSize: 0,
                        minChunks: 2
                    }
                }
            }
        }
    };
    ```

- 利用缓存提升二次加载速度
 - hard-source-webpack-plugin

#### 常见的loader有哪些，使用过哪些loader
- file-loader
- css-loader
- image-loader
- less-loader
- babel-loader
- ts-loader
- eslint-loader


#### 有哪些常用插件， 使用过哪些
- html-webpack-plugin 简化html创建
- mini-css-extract-plugin 提取css文件
- define-plugin 定义环境变量 (Webpack4 之后指定 mode 会自动配置)
- hardsource-webpack-plugin 启用缓存
- happypack 多进程构建
- webpack-bundle-analyzer  可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)
- clean-webpack-plugin

#### webpack中loader和plugin区别
<font color=orange>Loader</font> 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作    

<font color=orange>Plugin</font> 就是插件，基于事件流框架 Tapable，插件可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果        

<font color=orange>Loader</font> 在 module.rules 中配置，作为模块的解析规则，类型为数组。每一项都是一个 Object，内部包含了 test(类型文件)、loader、options (参数)等属性。

<font color=orange>Plugin</font> 在 plugins 中单独配置，类型为数组，每一项是一个 Plugin 的实例，参数都通过构造函数传入。  

#### webpack构建流程简述下
- 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数
- 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译
- 确定入口：根据配置中的 entry 找出所有的入口文件
- 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
- 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
- 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
- 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。       

简单说
- 初始化：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler
- 编译：从 Entry 出发，针对每个 Module 串行调用对应的 Loader 去翻译文件的内容，再找到该 Module 依赖的 Module，递归地进行编译处理
- 输出：将编译后的 Module 组合成 Chunk，将 Chunk 转换成文件，输出到文件系统中


#### webpack如何文件监听
webpack 开启监听有2种方式
- 启动命令行添加 --watch
- webpack 配置文件中 添加 watch: true

#### babel编译原理
- 解析：将代码转换成 AST
- 转换：访问 AST 的节点进行变换操作生产新的 AST
- 生成：以新的 AST 为基础生成代码


#### less-loader 源码
```javascript
    import less from 'less';
    import { getOptions } from 'loader-utils';
    import { validate } from 'schema-utils';
    import schema from './options.json';
    async function lessLoader(source) {
    const options = getOptions(this);
    //校验参数
    validate(schema, options, {
        name: 'Less Loader',
        baseDataPath: 'options',
    });
    const callback = this.async();
    //对options进一步处理，生成less渲染的参数
    const lessOptions = getLessOptions(this, options);
    //是否使用sourceMap，默认取options中的参数
    const useSourceMap =
        typeof options.sourceMap === 'boolean' 
        ? options.sourceMap : this.sourceMap;
    //如果使用sourceMap，就在渲染参数加入
    if (useSourceMap) {
        lessOptions.sourceMap = {
        outputSourceFiles: true,
        };
    }
    let data = source;
    let result;
    try {
        result = await less.render(data, lessOptions);
    } catch (error) {
    }
    const { css, imports } = result;
    //有sourceMap就进行处理
    let map =
        typeof result.map === 'string' 
        ? JSON.parse(result.map) : result.map;
    
    callback(null, css, map);
    }
    export default lessLoader;
```

#### webpack module/bundle/chunk关系
- module 在webpack中，一个文件对应一个module
- chunk: webpack根据文件引用顺序，生成chunk文件
- bundle: 最终输出的文件，与chunk可能是多对一关系

module就是没有编译的源文件，webpack根据文件引用关系，生成chunk文件，webpack处理好chunk后，生成bundle文件
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a86fef7f54c14db1a2e3eea2031cf9d8~tplv-k3u1fbpfcp-zoom-1.image">
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c616f7fd0c6b4ce4a9b8dfe0f937bb4d~tplv-k3u1fbpfcp-zoom-1.image">

```javascript
    index.bundle.js/ index.bundle.css 在index.js被引用，分别被打包进chunk 0 中，最后chunk 0 生成了2个bundle
```

#### css-loader原理
解析css文件 @import和@url语句， 处理css文件，并将结果返回一个js文件 
打包后得到了 bundle-css.js，其中有3个 modules：
- ./node_modules/css-loader/dist/runtime/api.js => css-loader 内部的api.js
- ./src/templates/loaders/test.css

来看看css-loader对a.css的编译输出：
```javascript
// css-loader输出

exports = module.exports = require("../../../node_modules/css-loader/lib/css-base.js")(false);

// imports
// 文件需要的依赖js模块，这里为空

// module
exports.push([ // 模块导出内容
  module.id, 
  ".src-components-Home-index__c--3riXS {\n  font-weight: bolder;\n}\n.src-components-Home-index__b--I-yI3 {\n  color: red;\n}\n.src-components-Home-index__a--3EFPE {\n  font-size: 16px;\n}\n", 
  ""
]); 

// exports
exports.locals = { // css-modules的类名映射
  "c": "src-components-Home-index__c--3riXS",
  "b": "src-components-Home-index__b--I-yI3",
  "a": "src-components-Home-index__a--3EFPE"
};
export.tostring = () => {console.log('原css文件中的样式(类名被转换了，代码可能也被压缩过)')}
```

可以理解为css-loader将a.css、b.css和c.css的样式内容以字符串的形式拼接在一起，并将其作为js模块的导出内容。

可以用下面伪代码表示
```javascript
  // css-loader 处理的css文件
  module.exports = {
    toString: () => "原css文件中的样式(类名被转换了，代码可能也被压缩过)",
    locals: {
      [原文件中写的类名等]: "被转换后的实际类名等"
    }
  }
```
  
##### 使用样式
目前虽然可以通过 import 导入 css 文件了，但是 html 还没有套用我们引入的样式。常用的可以衔接 css-loader 套用样式到 html 的方法有两种：
- style-loader
- mini-css-extract-plugin 从bundle中抽离css文件，并用link标签插入到文档中

##### style-loader
有了css-loader，style-loader的实现就很简单了。先通过css-loader把css文件转换成一个对象，假设叫content，该对象就是上面css-loader转换css文件后导出的对象。
然后简单粗暴的通过DOM操作将content中的样式插入到style标签中。最后将content.locals导出，方便用户使用类名、动画名等。


##### 参考文章
- [Webpack Loader 原理学习之 css-loader](https://archive.vincent0700.com/2020/05/07/052_Css_loader/)
- [css-loader原理](https://zhuanlan.zhihu.com/p/360552757)

#### webpack3升级到webpack4为什么会提升速度 优化了哪些点
- 升级node版本 >8.94, webpack使用了js最新的语法
- 缩小解析范围，使用resolve中，alias, includes, exclude，extensions
- 生产环境默认开启了很多代码优化，例如， minify, splitChunks。 设置 mode = "production"
- 自动注入process.env.NODE_ENV， 这样就不需要手动 DefinePlugin
- optimize.UglifyJsPlugin 废弃，由 optimization.minimize 替代，生产环境默认开启。
- commonChunkPlugin替换成splitChunks, CommonsChunkPlugin有以下缺点
  - CommonsChunkPlugin会提取我们不需要的代码
  - 在异步模块效率低
  - 很难使用，配置也很难理解
  - splitChunks对异步模块是开启的
- extract-text-webpack-plugin，替换成 mini-css-extract-plugin,并且要配置css-loader: 中loader指定为MiniCssExtractPlugin.loader
  - 异步加载
  - 不需要重复编译，性能更好
  - 更容易使用
  ```javascript
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            {
              loader: MiniCssExtractPlugin.loader,
              options: {
                publicPath,
              },
            },
            {
              loader: 'css-loader',
            },
          ],
        },
      ]
    },
    plugin: [
      new MiniCssExtractPlugin({
        filename: 'css/[contenthash:8].css',
        chunkFilename: 'css/[contenthash:8].css',
        ignoreOrder: true,
      })
    ]
  ```

[使用webpack4提升180%编译速度](http://louiszhai.github.io/2019/01/04/webpack4/)

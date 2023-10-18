## 概念
本质上，webpack 是一个用于现代 JavaScript 应用程序的 静态模块打包工具。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 __依赖图(dependency graph)__，
然后将你项目中所需的每一个模块组合成一个或多个 bundles，它们均为静态资源，用于展示你的内容。


## webpack中的loader和plugin有什么区别
##### loader
+ webpack 只能理解 JavaScript 文件。loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 模块。
+ 在module的rules数组里配置必须包含两个属性：test（识别出哪些文件会被转换）、use（定义出在进行转换时，应该使用哪个 loader）
+ 可以配置多个loader(从下到上，从左到右)，支持链式调用

##### 常用loader
+ css-loader、style-loader、postcss-loader、less-loader、sass-loader node-sass
+ file-loader：解决图片引入问题，并将图片 copy 到指定目录，默认为 dist
+ url-loader：当图片小于 limit 值的时候，会将图片转为 base64 编码，大于 limit 值的时候依然是使用 file-loader 进行拷贝
+ img-loader：压缩图片
+ babel-loader：加载 ES2015+ 代码并将其转换为 ES5
+ thread-loader：多进程配置
+ cache-loader：持久化缓存


##### plugin
+ 就是插件，插件可以扩展 Webpack 的功能，包括：打包优化，资源管理，注入环境变量。
+ require()引用，添加到plugins数组,插件可以携带参数/选项需向 plugins 属性传入一个 new 实例（插件可以携带参数/选项）。
+ webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且在 整个 编译生命周期都可以访问 compiler 对象。
```javascript
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';
class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建正在启动！');
    });
  }
}
module.exports = ConsoleLogOnBuildWebpackPlugin;
```
##### 常用插件
+ html-webpack-plugin：js 或者 css 文件可以自动引入到 Html 中
+ clean-webpack-plugin：自动清空打包目录
+ speed-measure-webpack-plugin：构建费时分析
+ webpack-bundle-analyzer：构建结果分析
+ optimize-css-assets-webpack-plugin:压缩css
+ terser-webpack-plugin：压缩JS

## 其他配置
##### 启动 devServer
```javascript
devServer: {
  // 在设置 contentBase 之后，就直接到对应的静态目录下面去读取文件，而不需对文件做任何移动，节省了时间和性能开销。
  contentBase: path.resolve(__dirname, 'public'), // 静态文件目录
  compress: true, //是否启动压缩 gzip
  port: 8080, // 端口号
  // open:true  // 是否自动打开浏览器
}
```
##### SourceMap 配置选择
SourceMap 是一种映射关系，当项目运行后，如果出现错误，我们可以利用 SourceMap 反向定位到源码位置

##### 三种 hash 值
+ hash ：任何一个文件改动，整个项目的构建 hash 值都会改变；
+ chunkhash：文件的改动只会影响其所在 chunk 的 hash 值；
+ contenthash：每个文件都有单独的 hash 值，文件的改动只会影响自身的 hash 值

## webpack工作流程
+ 入口文件（Entry）：Webpack 的工作开始于一个或多个入口文件，这些文件通常是你应用程序的主要入口点
+ 依赖分析（Dependency Analysis）：Webpack 开始分析入口文件，并识别所有的依赖模块。这包括直接依赖（如 import 语句引入的模块）和间接依赖
+ 加载器（Loaders）：对于不同类型的文件（例如，JavaScript、CSS、图片），Webpack 使用加载器来将它们转换为可管理的模块。加载器是用于在加载模块时对其进行预处理的工具
+ 生成依赖图（Dependency Graph）：Webpack 构建一个包含所有依赖关系的依赖图。这个图形确定了模块之间的依赖关系和加载顺序。
+ 插件（Plugins）：Webpack 使用插件来执行各种任务，例如生成 HTML 文件、拷贝静态资源、压缩代码等。插件可以在构建流程中执行额外的操作。
+ 输出文件（Output）：Webpack 最终将所有模块和依赖关系打包到一个或多个输出文件中。这些输出文件通常包括 JavaScript 文件、CSS 文件和其他资源文件
+ 性能优化：Webpack 提供了各种性能优化选项，例如压缩代码、消除重复、提取共享代码等，以确保生成的包大小尽可能小，加载速度尽可能快
+ 输出到生产环境（Production Build）：在构建完成后，将生成的包文件部署到生产环境中

## webpack热更新
Webpack 的热更新又称热替换（Hot Module Replacement），缩写为 HMR。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。  
HMR的核心就是客户端从服务端拉去更新后的文件，准确的说是 chunk diff (chunk 需要更新的部分)，实际上 WDS（webpack-dev-server） 与浏览器之间维护了一个 Websocket，当本地资源发生变化时，WDS 会向浏览器推送更新，并带上构建时的 hash，让客户端与上一次资源进行对比。客户端对比出差异后会向 WDS 发起 Ajax 请求来获取更改内容(文件列表、hash)，这样客户端就可以再借助这些信息继续向 WDS 发起 jsonp 请求获取该chunk的增量更新。后续的部分由 HotModulePlugin 来完成

## 优化webpack的构建速度
+ thread-loader 多进程打包，可以大大提高构建的速度，使用方法是将thread-loader放在比较费时间的loader之前，比如babel-loader
+ 使用 cache-loader 启用持久化缓存。使用 package.json 中的 "postinstall" 清除缓存目录。
+ 使用高版本的 Webpack 和 Node.js
+ 压缩代码，图片压缩，Tree shaking；模块懒加载；Gzip；小图片转base64；合理配置hash
+ 开启热更新;exclude：不需要处理的文件;include：需要处理的文件
+ 构建区分环境，区分环境去构建是非常重要的，我们要明确知道，开发环境时我们需要哪些配置，不需要哪些配置；而最终打包生产环境时又需要哪些配置，不需要哪些配置：

## 如何写一个loader
+ Loader 支持链式调用，所以开发上需要严格遵循“单一职责”，每个 Loader 只负责自己需要负责的事情。
+ Webpack最后打包出来的是一份Javascript代码,Loader返回值必须是标准的JS代码字符串，以保证下一个loader能够正常工作
+ Loader 运行在 Node.js 中，我们可以调用任意 Node.js 自带的 API 或者安装第三方模块进行调用

## 如何写一个plugin
+ 插件需要在其原型上绑定apply方法，才能访问 compiler 实例
+ 传给每个插件的 compiler 和 compilation对象都是同一个引用，若在一个插件中修改了它们身上的属性，会影响后面的插件
+ 异步的事件需要在插件处理完任务时调用回调函数通知 Webpack 进入下一个流程，不然会卡住

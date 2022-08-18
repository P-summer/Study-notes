## 概念
本质上，webpack 是一个用于现代 JavaScript 应用程序的 静态模块打包工具。当 webpack 处理应用程序时，它会在内部从一个或多个入口点构建一个 __依赖图(dependency graph)__，
然后将你项目中所需的每一个模块组合成一个或多个 bundles，它们均为静态资源，用于展示你的内容。


## webpack中的loader和plugin有什么区别
##### loader
+ webpack 只能理解 JavaScript 文件。loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 模块。
+ 在module的rules数组里配置必须包含两个属性：test（识别出哪些文件会被转换）、use（定义出在进行转换时，应该使用哪个 loader）
+ 可以配置多个loader(从下到上，从左到右)，支持链式调用
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

## webpack工作流程
+ 初始化参数：校验配置文件，初始化本次构建的配置参数
+ 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译
+ 确定入口：根据配置的entry找到所有的入口
+ 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
+ 完成模块编译：在使用loader编译完所有模块后，得到每个模块被编译后的最终内容已经之间所依赖的关系
+ 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
+ 输出完成：在确认好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入文件系统

## webpack热更新
Webpack 的热更新又称热替换（Hot Module Replacement），缩写为 HMR。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。  
HMR的核心就是客户端从服务端拉去更新后的文件，准确的说是 chunk diff (chunk 需要更新的部分)，实际上 WDS（webpack-dev-server） 与浏览器之间维护了一个 Websocket，当本地资源发生变化时，WDS 会向浏览器推送更新，并带上构建时的 hash，让客户端与上一次资源进行对比。客户端对比出差异后会向 WDS 发起 Ajax 请求来获取更改内容(文件列表、hash)，这样客户端就可以再借助这些信息继续向 WDS 发起 jsonp 请求获取该chunk的增量更新。后续的部分由 HotModulePlugin 来完成

## 优化webpack的构建速度
+ 使用高版本的 Webpack 和 Node.js
+ 每个额外的 loader/plugin 都有启动时间。尽量少使用不同的工具。
+ 压缩代码，图片压缩，Tree shaking
+ 使用 cache-loader 启用持久化缓存。使用 package.json 中的 "postinstall" 清除缓存目录。
+ 使用 DllPlugin 将更改不频繁的代码进行单独编译。改善引用程序的编译速度

## 如何写一个loader
+ Loader 支持链式调用，所以开发上需要严格遵循“单一职责”，每个 Loader 只负责自己需要负责的事情。
+ Webpack最后打包出来的是一份Javascript代码,Loader返回值必须是标准的JS代码字符串，以保证下一个loader能够正常工作
+ Loader 运行在 Node.js 中，我们可以调用任意 Node.js 自带的 API 或者安装第三方模块进行调用

## 如何写一个plugin
+ 插件需要在其原型上绑定apply方法，才能访问 compiler 实例
+ 传给每个插件的 compiler 和 compilation对象都是同一个引用，若在一个插件中修改了它们身上的属性，会影响后面的插件
+ 异步的事件需要在插件处理完任务时调用回调函数通知 Webpack 进入下一个流程，不然会卡住

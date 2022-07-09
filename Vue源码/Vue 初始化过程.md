## Vue 初始化过程
实例化Vue的时候，会调用init方法，init方法主要的操作包括：
+ 合并配置项  
  组件初始化会做一些性能优化的处理，将构造函数的配置对象放到$opation上，减少运行时原型链的查找，提高执行效率  
  选项合并，合并 Vue 的全局配置到根组件的局部配置
+ 初始化组件实例关系属性：$parent,$children,$refs,$root
+ 处理自定义事件，事件的派发和监听都是事件本身
+ initRender,处理渲染函数，解析组件的插槽信息，得到 vm.$slot，，vm._c它是被模板编译成的 render 函数使用,vm.$createElement是用户手写 render 方法使用的
+ 调用beforeCreate钩子函数
+ 初始化inject配置项，返回一个result[key] = val 配置对象，对数据进行响应式处理
+ initState 数据相应，处理 props、methods、data、computed、watch
+ 解析provide对象 将其挂载到 vm._provided 属性上
+ 调用 created 钩子函数 到这里可以看出我们在beforecreate之前是获取不到数据的，最先能获取数据的生命周期就是create
+ 挂载阶段  有 el 项，自动挂载，没有手动挂载

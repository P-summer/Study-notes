### Runtime Only VS Runtime + Compiler  
通常我们利用 vue-cli 去初始化我们的 Vue.js 项目的时候会询问我们用 Runtime Only 版本的还是 Runtime + Compiler 版本。  
+ __Runtime Only__  
我们在使用 Runtime Only 版本的 Vue.js 的时候，通常需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，
所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。  
+ __Runtime + Compiler__  
我们如果没有对代码做预编译，但又使用了 Vue 的 template 属性并传入一个字符串，则需要在客户端编译模板。  
```javascript
// 需要编译器的版本
new Vue({
  template: '<div>{{ hi }}</div>'
})
// 这种情况不需要
new Vue({
  render (h) {
    return h('div', this.hi)
  }
})
```
因为在 Vue.js 2.0 中，最终渲染都是通过 render 函数，如果写 template 属性，则需要编译成 render 函数，那么这个编译过程会发生运行时，所以需要带有编译器的版本。  
很显然，这个编译过程对性能会有一定损耗，所以通常我们更推荐使用 Runtime-Only 的 Vue.js。

### vue实例挂载的实现
Vue 中我们是通过 $mount 实例方法去挂载 vm 的，$mount 方法在多个文件中都有定义，如 src/platform/web/entry-runtime-with-compiler.js、
src/platform/web/runtime/index.js、src/platform/weex/runtime/index.js。因为 $mount 这个方法的实现是和平台、构建方式都相关的。  

__compiler 版本的 $mount 实现非常有意思，先来看一下 src/platform/web/entry-runtime-with-compiler.js 文件中定义：__  
这段代码首先缓存了原型上的 $mount 方法，再重新定义该方法，我们先来分析这段代码。首先，它对 el 做了限制，Vue 不能挂载在 body、
html 这样的根节点上。接下来的是很关键的逻辑 —— __如果没有定义 render 方法，则会把 el 或者 template 字符串转换成 render 方法__。
这里我们要牢记，在 Vue 2.0 版本中，所有 Vue 的组件的渲染最终都需要 render 方法，无论我们是用单文件 .vue 方式开发组件，
还是写了 el 或者 template 属性，最终都会转换成 render 方法，那么这个过程是 Vue 的一个“在线编译”的过程，它是调用
__compileToFunctions__ 方法实现的，编译过程我们之后会介绍。最后，调用原先原型上的 $mount 方法挂载。  

__原先原型上的 $mount 方法在 src/platform/web/runtime/index.js 中定义，之所以这么设计完全是为了复用，因为它是可以被 runtime only 版本的 Vue 直接使用的。__
```javascript
// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
$mount 方法支持传入 2 个参数，第一个是 el，它表示挂载的元素，可以是字符串，也可以是 DOM 对象，如果是字符串在浏览器环境下会调用 query 方法转换成 DOM 对象的。
第二个参数是和服务端渲染相关，在浏览器环境下我们不需要传第二个参数。  
$mount 方法实际上会去调用 ```mountComponent``` 方法，这个方法定义在 src/core/instance/lifecycle.js 文件中  
```mountComponent``` 核心就是先实例化一个渲染Watcher，在它的回调函数中会调用 updateComponent 方法，在此方法中调用 ```vm._render``` 方法先生成虚拟 Node，
最终调用 ```vm._update``` 更新 DOM。  
函数最后判断为根节点的时候设置 ```vm._isMounted``` 为 true， 表示这个实例已经挂载了，同时执行 mounted 钩子函数。 这里注意 vm.$vnode 表示 Vue 实例的父虚拟 Node，
所以它为 Null 则表示当前是根 Vue 的实例。

### 深入理解 Vue 的 patch 阶段，理解其 diff 算法的原理
Vue 的 ```_update``` 是实例的一个私有方法，它被调用的时机有 2 个，一个是首次渲染，一个是数据更新的时候
```javascript
/**
*  mountComponent 核心就是先实例化一个渲染Watcher
*  在它的回调函数中会调用 updateComponent 方法，在此方法中调用 vm._render 方法先生成虚拟 Node，最终调用 vm._update 更新 DOM。
*/
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}

// src/core/instance/lifecycle.js  
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
  // oldNode不存在，初始化渲染
  if (!prevVnode) {
    // initial render
    vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
  } else {
    // updates
    vm.$el = vm.__patch__(prevVnode, vnode)
  }
}

/**
*  _update 的核心就是调用 vm.__patch__ 方法,在不同的平台定义不一样
*  在浏览器端渲染中，它指向了 patch 方法,定义在 src/platforms/web/runtime/patch.js
*  传入了一个对象，包含 nodeOps 参数和 modules 参数。nodeOps 封装了一系列 DOM 操作的方法，modules 定义了一些模块的钩子函数的实现
*  createPatchFunction 的实现，它定义在 src/core/vdom/patch.js  
*  工厂函数，注入平台特有的一些功能操作，并定义一些方法，然后返回 patch 函
*/
Vue.prototype.__patch__ = inBrowser ? patch : noop

export const patch: Function = createPatchFunction({ nodeOps, modules })

/** 
* patch接收四个参数
* oldVnode 表示旧的 VNode 节点，vnode 表示执行 _render 后返回的 VNode 的节点
* hydrating 表示是否是服务端渲染；removeOnly 是给 transition-group 用的
*/
return function patch (oldVnode, vnode, hydrating, removeOnly) {
// 如果新节点不存在，老节点存在，则调用 destroy，销毁老节点
  if (isUndef(vnode)) {
    if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
    return
  }
  // 新节点存在，老节点不存在，这种情况会在一个组件初次渲染的时候出现
  if (isUndef(oldVnode)) {
      // empty mount (likely as component), create new root element
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
    } else {
      // 判断 oldVnode 是否为真实元素
      const isRealElement = isDef(oldVnode.nodeType)
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // 不是真实元素，但是老节点和新节点是同一个节点，则是更新阶段，执行 patch 更新节点
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
      } else {
        // 是真实元素，则表示初次渲染
        if (isRealElement) {
          // 挂载到真实元素以及处理服务端渲染的情况
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
            oldVnode.removeAttribute(SSR_ATTR)
            hydrating = true
          }
          if (isTrue(hydrating)) {
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              invokeInsertHook(vnode, insertedVnodeQueue, true)
              return oldVnode
            } else if (process.env.NODE_ENV !== 'production') {
              warn(
                'The client-side rendered virtual DOM tree is not matching ' +
                'server-rendered content. This is likely caused by incorrect ' +
                'HTML markup, for example nesting block-level elements inside ' +
                '<p>, or missing <tbody>. Bailing hydration and performing ' +
                'full client-side render.'
              )
            }
          }
          // 走到这儿说明不是服务端渲染，或者 hydration 失败，则根据 oldVnode 创建一个 vnode 节点
          // either not server-rendered, or hydration failed.
          // create an empty node and replace it
          oldVnode = emptyNodeAt(oldVnode)
        }
        // 拿到老节点的真实元素
        const oldElm = oldVnode.elm
        // 获取老节点的父元素，即 body
        const parentElm = nodeOps.parentNode(oldElm)
        // 基于新 vnode 创建整棵 DOM 树并插入到 body 元素下
        createElm(
          vnode,
          insertedVnodeQueue,
          // extremely rare edge case: do not insert if old element is in a
          // leaving transition. Only happens when combining transition +
          // keep-alive + HOCs. (#4590)
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        )
        // 递归更新父占位符节点元素
        if (isDef(vnode.parent)) {
        }
        // 移除老节点
        if (isDef(parentElm)) {
          removeVnodes([oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode)
        }
      }
   }
}
// 基于 vnode 创建整棵 DOM 树，并插入到父节点上
function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {
    /**
    *  如果 vnode 是一个组件，则执行 init 钩子，创建组件实例并挂载,为组件执行各个模块的 create 钩子
    *  如果组件被 keep-alive 包裹，则激活组件;如果是一个普通元素，则什么也不做
    */
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }
    // 获取VNode的data对象和孩子节点
    const data = vnode.data
    const children = vnode.children
    const tag = vnode.tag
    if (isDef(tag)) {
      // 未知标签
      // 创建新节点
      vnode.elm = vnode.ns
        ? nodeOps.createElementNS(vnode.ns, tag)
        : nodeOps.createElement(tag, vnode)
      setScope(vnode)

      // 递归创建所有子节点（普通元素、组件）
      createChildren(vnode, children, insertedVnodeQueue)
      if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue)
      }
      // 将节点插入父节点
      insert(parentElm, vnode.elm, refElm)
    } else if (isTrue(vnode.isComment)) {
      // 注释节点，创建注释节点并插入父节点
      vnode.elm = nodeOps.createComment(vnode.text)
      insert(parentElm, vnode.elm, refElm)
    } else {
      // 文本节点，创建文本节点并插入父节点
      vnode.elm = nodeOps.createTextNode(vnode.text)
      insert(parentElm, vnode.elm, refElm)
    }
  }


```


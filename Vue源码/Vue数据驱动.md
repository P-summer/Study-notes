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

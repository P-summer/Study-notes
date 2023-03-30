### 为什么vue3 比vue2性能快了不少，在哪方面做了优化

Vue3相对于Vue2在性能上有较大的提升，主要是因为下面几个方面的优化：

1. 更快的虚拟DOM算法：Vue3使用了更快的虚拟DOM算法，将渲染性能提高了很多。Vue3的虚拟DOM算法使用了静态标记和缓存，可以在运行时避免重复计算。

2. 更小的运行时包：Vue3的运行时包比Vue2更小，减少了加载时间和内存使用。Vue3去除了一些不常用的功能和指令，使得运行时包更加轻量级。

3. 更好的编译器优化：Vue3的编译器比Vue2更加智能化，能够在编译时对代码进行更好的优化。Vue3使用了基于模板的静态分析和编译优化，减少了运行时的开销。

4. 更好的组件复用性：Vue3支持更好的组件复用性，使得开发者可以更轻松地重用组件并提高代码的可维护性。Vue3的Composition API让组件逻辑更容易被抽象和复用。

5. 更灵活的响应式系统：Vue3的响应式系统比Vue2更灵活，使得开发者可以更好地控制组件的状态和数据流。Vue3的响应式系统支持Proxy代理对象，可以更细粒度地控制数据的访问和修改。

综上所述，Vue3在虚拟DOM算法、运行时包大小、编译器优化、组件复用性和响应式系统等方面都做了较大的优化，从而使得性能得到了显著提升。



### vue3和react的区别是什么，体现在几个方面

Vue3和React都是流行的JavaScript框架，它们有一些相似之处，但也有一些不同之处。下面是Vue3和React的区别：

1. 模板语法：Vue3使用了类似于HTML的模板语法，而React使用了JSX语法。Vue3的模板语法更加直观易懂，可以更快速地开发。

2. 组件复用性：Vue3的Composition API使得组件逻辑更容易被抽象和复用，而React使用了高阶组件和render props来实现组件的复用。

3. 响应式系统：Vue3的响应式系统支持Proxy代理对象，可以更细粒度地控制数据的访问和修改，而React使用了单向数据流和不可变数据来处理数据的变化。

4. 集成度：Vue3集成了Vue Router和Vuex等多个插件，而React需要使用第三方库来实现路由、状态管理等功能。

5. 性能：Vue3在性能方面做了很多优化，如更快的虚拟DOM算法、更小的运行时包等，而React则需要使用PureComponent和shouldComponentUpdate等手段来优化性能。


### 详细说一下 Vue2 和 Vue3 diff 算法的详细过程。这两个算法的复杂度。
Vue2 和 Vue3 的 diff 算法都是基于 Virtual DOM 的，但是在实现上有一些区别。

Vue2 的 diff 算法

Vue2 的 diff 算法采用的是双指针的方式，即分别有 oldStartIdx、oldEndIdx、newStartIdx、newEndIdx 四个指针分别指向旧的子节点的开始和结束位置以及新的子节点的开始和结束位置。在比较过程中，分为四种情况：

1. oldStartVnode 和 newStartVnode 相同，直接进行 patch，oldStartIdx 和 newStartIdx 分别加 1。

2. oldEndVnode 和 newEndVnode 相同，直接进行 patch，oldEndIdx 和 newEndIdx 分别减 1。

3. oldStartVnode 和 newEndVnode 相同，直接进行 patch，oldStartIdx 加 1，newEndIdx 减 1。

4. oldEndVnode 和 newStartVnode 相同，直接进行 patch，oldEndIdx 减 1，newStartIdx 加 1。

如果四种情况都不满足，则遍历旧子节点，尝试在旧子节点中找到与 newStartVnode 相同的节点，如果找到了，则进行 patch，同时将旧子节点中对应的位置设置为 undefined，表示已经找到了。如果没有找到，则说明 newStartVnode 是一个新节点，直接创建并插入到旧子节点的开头位置。

遍历完成后，如果 oldStartIdx > oldEndIdx，说明旧子节点已经全部处理完了，此时将新子节点中剩余的节点都插入到旧子节点的末尾；如果 newStartIdx > newEndIdx，说明新子节点已经全部处理完了，此时将旧子节点中剩余的节点都删除。

Vue2 的 diff 算法时间复杂度为 O(n^2)，因为需要遍历旧子节点来寻找与新子节点相同的节点，如果旧子节点很多，这个过程会非常耗时。

Vue3 的 diff 算法

Vue3 的 diff 算法采用的是双端比较的方式，即同时从新旧子节点的头部和尾部开始遍历，同时从新子节点的头部和尾部开始遍历  
从左开始对比，相等patch i+1  
从右开始对比，相等 e1-1,e2-1  
判断e1,e2和i的大小关系，e1>e2删除，e2>e1 新增  
左右两边都比对完了，然后剩下的就是中间部位顺序变动的，最长递增子序列

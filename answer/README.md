##### 1.Vue3.0性能提升主要是通过哪几方面体现的？

1. 响应式系统的提升

> vue2.x在初始化的时候，会对data中的每个属性都使用defineProperty调用getter和setter来变成响应式对

象。如果属性值为对象，还会递归调用defineProperty使之变成响应式对象

> vue3.0使用proxy对象重写响应式。proxy的性能比defineProperty要好，proxy可以拦截属性的访问、赋值、

删除等操作，不需要初始化的时候遍历所有属性，另外有多层属性嵌套的化，只有访问某个属性的时候，才会递归

处理下一级的属性

2. 编译优化

> 优化编译和重写虚拟dom，让首次渲染和更新dom性能有更大的提升,vue2.x通过标记静态节点，优化diff算法

  vue3.0标记和提升静态根节点,diff的时候只比较动态节点的内容

> Fragments模板里不用创建唯一根节点，可以直接放同级标签和文本内容

> 静态提升

> patch flag 跳过静态节点,直接对比动态节点

> 缓存事件处理函数

3. 源码体积的优化

> vue3.0移除了一些不常用的api

> 使用tree-shaking
##### 2.Vue3.0所采用的Composition Api与Vue2.x使用的Options Api有什么区别？

> Options API 以vue为后缀的文件，通过定义methods,computed,watch,data等属性与方法，共同处理页面逻

  辑，Options代码编写的方式，如果是组件状态，写在data属性上，如果是方法，写在methods属性上，使

  组件变得更加复杂，导致对应属性列表也会增长，这可能会导致组件难以阅读和理解

> Composition API，根据逻辑功能来组织，一个功能所定义的所有API都会放在一起，高内聚，低耦合

> 总结 
  
  在逻辑组织和逻辑复用方面，Composition API是优于Options API

  因为Composition API几乎是函数，会有更好的类型推断

  Composition API 对tree-shaking友好，代码也容易压缩

  Compositon API 中见不到this的使用，减少了this指向不明的情况

  如果是小型组件，可以继续使用Options API

##### 3.Proxy相对于Object defineProperty有哪些优点？

> Proxy可以直接监听对象而非属性

> Proxy可以直接监听数组变化

> Proxy有多达13种拦截方法，不限于apply ownKeys deleteProperty has等

> Proxy返回的是一个新对象，我们可以只操作新的对象而达到目的，而Object.defineProperty只能遍历对象

  属性直接修改

##### 4.Vue3.0在编译方面有哪些优化？

> vue.js 3.x中标记和提升所有的静态节点，diff的时候只需要对比动态节点内容；

> Fragments（升级vetur插件): template中不需要唯一根节点，可以直接放文本或者同级标签

> patch flag, 在动态标签末尾加上相应的标记,只能带 patchFlag 的节点才被认为是动态的元素,会被追踪属

 性的修改,能快速的找到动态节点,而不用逐个逐层遍历，提高了虚拟dom diff的性能。

> 缓存事件处理函数cacheHandler,避免每次触发都要重新生成全新的function去更新之前的函数

> tree shaking 通过摇树优化核心库体积,减少不必要的代码量

##### 5.Vue.js3.0响应式系统的实现原理？

1. reactive
设置对象为响应式对象。接收一个参数 判断这参数是否是对象。不是对象则直接返回这个参数，不做响应式处理。
创建拦截器handerler，设置get/set/deleteproperty
get
收集依赖（track）；
如果当前 key 的值是对象，则为当前 key 的对象创建拦截器 handler, 设置 get/set/deleteProperty；
如果当前的 key 的值不是对象，则返回当前 key 的值。
set
设置的新值和老值不相等时，更新为新值，并触发更新（trigger）。
deleteProperty
当前对象有这个 key 的时候，删除这个 key 并触发更新（trigger）。

2. effect
接收一个函数作为参数。作用是：访问响应式对象属性时去收集依赖

3. track
接收两个参数：target 和 key
－如果没有 activeEffect，则说明没有创建 effect 依赖
－如果有 activeEffect，则去判断 WeakMap 集合中是否有 target 属性
－WeakMap 集合中没有 target 属性，则 set(target, (depsMap = new Map()))
－WeakMap 集合中有 target 属性，则判断 target 属性的 map 值的 depsMap 中是否有 key 属性
－depsMap 中没有 key 属性，则 set(key, (dep = new Set()))
－depsMap 中有 key 属性，则添加这个 activeEffect

4. trigger
判断 WeakMap 中是否有 target 属性，WeakMap 中有 target 属性，则判断 target 属性的 map 值中是否有 key 属性，有的话循环触发收集的 effect()。


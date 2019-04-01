_render的方法解析
---

Vue 的 `_render` 方法是实例的一个私有方法，它用来把实例渲染成一个虚拟 `Node`。

首先`_render`方法是在初始化的时候文件`src/core/instance/index.js`Vue初始化的文件中定义的。

```javascript
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
```
***代码 1.1***

其中`代码 1.1`中的`renderMixin`初始化对应的文件 `src/core/instance/render.js`文件中`Vue.prototype._render`方法是本次分析的重点

我们一起来看看这段代码：

```javascript
Vue.prototype._render = function (): VNode {
    const vm: Component = this
    const {
      render,
      staticRenderFns,
      _parentVnode
    } = vm.$options

    if (vm._isMounted) {
      // if the parent didn't update, the slot nodes will be the ones from
      // last render. They need to be cloned to ensure "freshness" for this render.
      for (const key in vm.$slots) {
        const slot = vm.$slots[key]
        if (slot._rendered) {
          vm.$slots[key] = cloneVNodes(slot, true /* deep */)
        }
      }
    }

    vm.$scopedSlots = (_parentVnode && _parentVnode.data.scopedSlots) || emptyObject

    if (staticRenderFns && !vm._staticTrees) {
      vm._staticTrees = []
    }
    // set parent vnode. this allows render functions to have access
    // to the data on the placeholder node.
    vm.$vnode = _parentVnode
    // render self
    let vnode
    try {
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      handleError(e, vm, `render function`)
      // return error render result,
      // or previous vnode to prevent render error causing blank component
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production') {
        vnode = vm.$options.renderError
          ? vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
          : vm._vnode
      } else {
        vnode = vm._vnode
      }
    }
    // return empty vnode in case the render function errored out
    if (!(vnode instanceof VNode)) {
      if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
          'should return a single root node.',
          vm
        )
      }
      vnode = createEmptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    return vnode
  }
```
***代码 1.2***

我们可以看到很多关于`$scopedSlots`、`$slots`、`$options`等，后续会分析各自的意义。本次主要看代码片段1.3中

```javascript
const {
      render,
      staticRenderFns,
      _parentVnode
    } = vm.$options
```
***代码 1.3***

其中`render`是从`vm.$options`中获取。具体`vm.$options`中如何定义，可以自行查看初始化的而过程。

之后代码会执行到`vnode = render.call(vm._renderProxy, vm.$createElement)`的片段

> vm._renderProxy 是什么？

这是最为关键的两个参数，前面已经分析过了`vm._renderProxy`分为2个方式，生产环境下就是`vm`的意思，如果是开发环境则是
如下代码片段`src/core/instance/init.js`中

```javascript
 /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
```

***代码 1.4***

可以看到`initProxy`就是在开发环境才会进入使用。然后可以进入代码片段分析`src/core/instance/proxy.js`中。最终主要就是在开发环境中，提前报出一些常见错误，基本都是
在这里报出的，比如模版参数不正确等。


> vm.$createElement 是什么

我们可以看到`render.js`文件中的最开始的定义 方法`initRender`这个是在初始化的时候调用的。

可以看到预定义的几个静态变量

```javascript
const parentVnode = vm.$vnode = vm.$options._parentVnode // the placeholder node in parent tree
const renderContext = parentVnode && parentVnode.context
```

***代码 1.5***

其实就是获取父节点和文本的对象，再往下查看代码

```javascript
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```

***代码 1.6***

上面的两行代码都是使用了`createElement`的方法。

>> 区别就在于，前者是给有编译环境下使用的render方法，后者是给手写render方法使用的

这篇主要介绍到这里。关于createElement参数可以查看官网[createElemnt](https://cn.vuejs.org/v2/guide/render-function.html#%E5%AE%8C%E6%95%B4%E7%A4%BA%E4%BE%8B)

### 总结

`vm._render` 最终是通过执行 `createElement` 方法并返回的是 `vnode`，它是一个虚拟 `Node`。Vue 2.0 相比 Vue 1.0 最大的升级就是利用了 `Virtual DOM`。因此在分析 `createElement` 的实现前，我们先了解一下 `Virtual DOM` 的概念










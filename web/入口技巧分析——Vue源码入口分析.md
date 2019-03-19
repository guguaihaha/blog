第一章：入口技巧分析篇

---

> 如何初始化全局命名空间前缀

其实细心可以根据引用路径发现文件`src/core/instance/index.js`开始。万恶之源的起始之地：

```javascript
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```
为啥不用class呢？一开始我也疑惑，后来才发现，后面还有文章

```javascript
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)
```

我***，这么多混入，随便打开一个看看啥玩意吧

```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    ...
  }
}
```
原来就是想神秘的在原型上挂点东西。其实到这里可以看到手法已经很高明了，不过作者本人认为还有可以修改的空间。

直接class初始化，通过内部`constructor`和`import`结合来初始化各个模块单元，当然也可以传入this到各个模块中自行挂载，好了继续看源码。

再看看`stateMixin`

```javascript
export function stateMixin (Vue: Class<Component>) {
  
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)

  Vue.prototype.$set = set
  Vue.prototype.$delete = del

  Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    ...
  }
}
```
还是一样啊，就是在原型上挂载一堆堆的方法。呦西，继续往下走。


返回查找路径节点`src/core/index.js`

啊哈，这里面有个`initGlobalAPI(Vue)` 这是干啥的呢，一起来看看吧。

```javascript
function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  Vue.options = Object.create(null)
  
  ASSET_TYPES.forEach(type => {
      Vue.options[type + 's'] = Object.create(null)
    })
  // this is used to identify the "base" constructor to extend all plain-object
    // components with in Weex's multi-instance scenarios.
    Vue.options._base = Vue
  
    extend(Vue.options.components, builtInComponents)
  
    initUse(Vue)
    initMixin(Vue)
    initExtend(Vue)
    initAssetRegisters(Vue)
}
```

呃，内容有点多，不过可以看到，里面第一就是config的配置。Vue的全局config配置（src/core/config.js），具体参考全局API查看吧。

同时还定义了`util`工具类（据说不稳定，还是别用，就是内部Vue自己用的，暴露出来就是玩玩）。还有 `set`、`delete`、`nexTick`、`options`等等

其中`ASSET_TYPES`吸引了我的目光，点进去查看

```javascript
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
```
哦，就是往`Vue.options`中添加对应的方法类型。干嘛弄的那么神秘，不过可以借鉴学习，不重要的模块变量或者变动比较大的单独抽离，方便变化。因为
大的项目开发中 **`多用组合少用继承和顺序写法`** 


再往下看，其中`Vue.options._base = Vue`留在后面章节分析

再往下看，`extend(Vue.options.components, builtInComponents)`中 `builtInComponents`是个组件呢，里面是啥我很好奇

```javascript
export default {
  KeepAlive
}
```

就是keepAlive组件啊,哇塞。内置的组件就是在这么悄无声息的植入啊。行了。到这里后续还有创建`use`，`mixin`，`extend`，`ASSET一些方法`。

然后回到主入口`src/platforms/entry-runtime-width-compiler.js`中查看一下片段：

> Vue.prototype.$mount

入口上来就是`$mount`的方法，这个方法干什么用的呢看看官方用法说明吧

```text
如果 Vue 实例在实例化时没有收到 el 选项，则它处于“未挂载”状态，没有关联的 DOM 元素。可以使用 vm.$mount() 手动地挂载一个未挂载的实例。

如果没有提供 elementOrSelector 参数，模板将被渲染为文档之外的的元素，并且你必须使用原生 DOM API 把它插入文档中。

这个方法返回实例自身，因而可以链式调用其它实例方法。
```

说白了一般用不上，只是分析时候从这里开始。不过有趣的是如下代码片段

```javascript
el = el && query(el)

/* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }
```

这里说明`el`是干啥的呢？通过分析可知，就是querySelector的选择器，如果没有则认为是元素本身，直接返回即可

```javascript
function query (el: string | Element): Element {
  if (typeof el === 'string') {
    const selected = document.querySelector(el)
    if (!selected) {
      process.env.NODE_ENV !== 'production' && warn(
        'Cannot find element: ' + el
      )
      return document.createElement('div')
    }
    return selected
  } else {
    return el
  }
}
```

到此时，还是继续分析`$mount`的内部逻辑可以注意参考Vue的api和业务方法。



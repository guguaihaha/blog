vue初始化之this的巧妙用法分析——vue源码分析
---

这篇主要揭晓一点初始化的过程中`this`的一些巧妙用法

上来看一个大家比较熟悉的简单demo

这次采用的是基于webpack打包和编译的流程（Runtime Only），不是实时编译(Runtime and Compiler)。具体查看前面介绍的区别。

好了，上来简单介绍这个`demo.vue`文件

```javascript

@import Vue from 'vue'

var app = new Vue({
  el:'#appId',
  mounted(){
      console.log(this.name)
  },
  data(){
      return{
          name: 'Hello World!'
      }
  }
})

```

请问`mounted`中输出数值`this.name`的含义是？

肯定很多人会说就是`Hello World!`，那请问如何取得的这个数值？不要直接意会就应该是这个，具体原因一起剖析一下吧。

首先回到Vue源代码库中的`src/core/instance/init.js`中，找到`Vue.prototype._init`这个方法。

其中细心的朋友可以发现方法的底部有很多初始化函数

```javascript
    ...
/* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    ...
```
其中我们一起看看`initState(vm)`这个方法。一起找到文件`src/core/instance/state.js`。

```javascript
function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

到这里我们看看上面的代码中有个`opts.data`判断语句中`initData(vm)`，这里就是关键点

看看`initData`中有什么好玩的吧。

```javascript
let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
```

哇哦，好多代码，不过咱们慢慢来分析，上来先搞清楚第一行`let data = vm.$options.data`
当然是获取了咱们前面定义的`demo.vue`文件中的data数据了，此时看后面会判断当前data是否是`function`类型，
当然这也是官方建议使用的返回类型。如果是函数类型，会进入到`getData（data，vm）`这个方法中。

哦，我们一起看看这个`getData`干嘛用的吧

```javascript
function getData (data: Function, vm: Component): any {
  try {
    return data.call(vm)
  } catch (e) {
    handleError(e, vm, `data()`)
    return {}
  }
}
```
呃，原来就是讲函数`借用构造`一下。不懂得去查一下吧。好了我们继续回归主线继续分析

后面就是判断当前`data`如果不是对象则会报出一个警告

呃，好难受分析到这里，总之就是个对象`data`就是安全的

>> 这里这里...

继续分析，后面看注释也能简单明白一下`proxy data on instance`看来用代理绑定数据了。这是啥

看到代码段，分别取出来了`data`对象中的key和vm中`props`与`methods`，再往下看，原来是开始比对了。

如果`data`中的key和`props`与`methods`的key一样就会报错。提出警告。为啥不能定义一样的key呢？

答案就是最后所有的对象属性都会挂在到同一个对象下`$vm`。所以要我说干脆就取消`props`和`methods`让新手趟坑（作者本人也是亲历者）

>> 那是如何挂在到this或者$vm上的呢？

其实就看代码else下面`proxy`的方法。里面发生了什么呢？也是本文章重点哦。来一起看看代码吧：

在同一个文件`src/core/instance/state.js`页面中找到`proxy`方法

```javascript
function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```
通过这段可以得知，当`mounted`中调用`this.name`时候其实就是访问了`this._data.name`。这就是所谓的代理了。

个人认为这个和组合继承有异曲同工之妙。就是`借用构造`和`原型继承`的组合用法。我相信翻译成es5之后也是这么去解释的。

好了。到这里也就明白了`this`的访问没那简单的。

最后还有一段`observe(data, true /* asRootData */)`数据的响应式处理。那就继续关注我吧，后续剖析。


### 总结

看来分析一个主线程要拆分很多模块，没必要的模块暂时不用看哦，先了解咱们要做什么去分析。也可以学习Vue作者的模块化拆分的精妙之处。







Vue源码分析——各种watcher的关系图
---

相信很多人都很迷惑，Vue到底怎么来维护那么多的watcher和watcher的种类有哪些？

总的来说，从源码角度看，watcher是个公共类，不同的流程里面去new 那就是不同种类的watcher

+ computed watcher

+ user watcher

+ render watcher

### render watcher

先说明`render watcher`，很是简单，就是触发渲染的流程，也就是在mount初始化的过程中创建的

```javascript
function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  vm._watcher = new Watcher(vm, updateComponent, noop)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

看到最后代码就是创建了渲染watcher

### computed watcher

在vue init阶段其中判断当前对象computed是否有数值，如果有则进行如下

```javascript
function initComputed (vm: Component, computed: Object) {
  process.env.NODE_ENV !== 'production' && checkOptionType(vm, 'computed')
  const watchers = vm._computedWatchers = Object.create(null)
  // computed properties are just getters during SSR
  const isSSR = isServerRendering()

  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }

    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
}
```

这就是计算属性的watcher代码

### user watcher

同理，也是在init阶段，但是有些许不一样的是创建过程，代码如下

```javascript
function initWatch (vm: Component, watch: Object) {
  process.env.NODE_ENV !== 'production' && checkOptionType(vm, 'watch')
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```

他是一个一个去遍历创建，为什么呢？后续会分析computed 和 user 区别

```javascript
function createWatcher (
  vm: Component,
  keyOrFn: string | Function,
  handler: any,
  options?: Object
) {
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }
  if (typeof handler === 'string') {
    handler = vm[handler]
  }
  return vm.$watch(keyOrFn, handler, options)
}
```

Vue的所有公共方法都在init之前进行了加载，可以自行翻阅查看

```javascript
Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {}
    options.user = true
    const watcher = new Watcher(vm, expOrFn, cb, options)
    if (options.immediate) {
      cb.call(vm, watcher.value)
    }
    return function unwatchFn () {
      watcher.teardown()
    }
  }
```

以上就是几个类型的watcher的源码部分。

代码不容易直白表述，主要上原理图

![原理图](http://www.zhangjinglin.cn/images/reactive.png)

根据原理图就能知道各个watcher之间的联系了

**那么有个问题了，render watcher没有任何问题，那么user watcher和computed watcher有什么区别呢？**

直接上图

首先是`user watcher`

![user watcher](http://www.zhangjinglin.cn/images/userwatcher.png)

然后是 `computed watcher`

![computed watcher](http://www.zhangjinglin.cn/images/computedwatcher.png)





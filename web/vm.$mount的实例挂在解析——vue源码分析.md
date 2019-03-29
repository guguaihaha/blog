vm.$mount的实例挂在解析——vue源码分析
---

Hello ! 大家好，今天小编本人基于之前的源码分析后的的后续方法解读。本篇主要分析，Vue的实例挂在的过程。这也是它的核心流程

再解析之前，我们回顾一下知识点：webpack打包和编译的流程`（Runtime Only）`和`Runtime and Compiler` 具体内容请查看前面的Vue源码章节细看。

这次主要还是从入口文件为模式是`Runtime and Compiler`的开始分析，入口文件`src/platforms/web/entry-runtime-width-compiler.js`分析。

代码是 `Vue.prototype.$mount` 

```javascript
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(template, {
        shouldDecodeNewlines,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}
```
***Code : 1.1***

我们一起来分析这段挂在实例的代码，第一行`const mount = Vue.prototype.$mount`

> 为啥会临时存储呢？

因为这要从`src/platforms/runtime/index.js`说起，这个主要是给 `Runtime Only` 复用做的，因为这个模式下没有上面的业务逻辑。

那我们看一看这个`src/platforms/runtime/index.js`的$mount方法吧：

```javascript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
***Code : 1.2***

基本上就是判断当前是否是客户端浏览器，然后执行`mountComponent(this, el, hydrating)`

这个方法代码如下：

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
***Code : 1.3***

上来就是`vm.$el = el` 进行实际节点挂在所用。

紧跟着就是`vm.$options.render`这个方法不存在则生成一个对应的`render`方法。

其中`createEmptyVNode` `callHook` `updateComponent` 后续会讲到，这里略过。
总之这里就是没有render就造一个出来。render就是个最后的重要凭证。一切转模版转render

我们只是看看最为重要的`vm._watcher = new Watcher(vm, updateComponent, noop)` 这个方法

代码路径在`src/core/instance/observer/watcher.js`中`Watcher`方法

```javascript
...
if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
          ? undefined
          : this.get()
...    
```
***Code : 1.4***

其中 `this.getter`对应Watcher中的第二个参数`updateComponent`。最后`this.value`赋值过程中主动调用`this.get()`方法。

```javascript
 get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }
```

***Code : 1.5***

看到 ` value = this.getter.call(vm, vm)` 会主动调用`this.getter`,也就是会调用`updateComponent`的这个方法。


方法在`src/core/instance/lifecycle`里面执行，执行代码片段是：

```javascript
vm._update(vm._render(), hydrating)
```

***Code : 1.6***

这里面就是把之前好不容易的的各种形式的`template`、`#template`、手写`render`等统一转换的render方法与实际的DOM做diff与替换。


本意到这里就完结了，没想到后面还有一行代码

```javascript
vm._watcher = new Watcher(vm, updateComponent, noop)
```

***Code : 1.7***

这句话的意义就是不光这次初始化需要替换，还在后续的数据放生变化的时候进行相应的替换。需要观察模式介入。

好了到这里基本$mount方法分析完毕。


> 我们再回到Code 1.1 的代码片段

可以看到，后续的`Vue.prototype.$mount`的重新定义与`src/platforms/runtime/index.js`中的`$mount`是一样的。这里就不再解释代码含义了。

###  总结：

最后$mount的方法意义在于就是将模版生成render函数，然后再去同步即可。包含后续的观察者模式持续的同步数据和Dom的变化。

其中很多细节方法本章莫有具体分析，不影响整体的分析流程。不过后面章节会详细介绍相关的具体方法含义与意义。






> 
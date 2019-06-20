双向数据绑定的原理——Vue源码分析
---

很多人讨论说Vue的双向数据绑定就是数据劫持实现的，这个观点只是说了表象，那么具体如何实现的呢？还是从源码开始剖析

### 准备知识点

+ Object.defineProperty

+ Event


### ES6的数据收集和派发

首先我们都知道，在Vue初始化的时候会在全局注册一个`init`方法，这就是万恶之源的开始

然后我们会找到`initState(vm)`的初始化方法，进入到最后`initData`方法

```javascript
function initData (vm: Component) {
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
}
```

如上面代码所述，其中关于遍历检查重复和建立`proxy`互相访问这块不谈，主要看`observe(data, true /* asRootData */)`

```javascript
function observe (value: any, asRootData: ?boolean): Observer | void {
  if (!isObject(value)) {
    return
  }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    observerState.shouldConvert &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

在`observe中`会进行是否该对象已经存在getter或者setter和最为关键的建立`new Observer`方法

```javascript
class Observer {
    constructor (value: any) {
        this.value = value
        this.dep = new Dep()
        this.vmCount = 0
        def(value, '__ob__', this)
        if (Array.isArray(value)) {
          const augment = hasProto
            ? protoAugment
            : copyAugment
          augment(value, arrayMethods, arrayKeys)
          this.observeArray(value)
        } else {
          this.walk(value)
        }
      }
  }
```
以上只是截取代码部分，我们看构造函类中的` this.observeArray(value)`进行递归调用和直接执行的`this.walk(value)`

```javascript
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }
```
遍历并执行`defineReactive`，因为要把所有的对象变成响应式的就要这么做。当然上面会有如果是数组怎么去处理的逻辑，感兴趣的自行查看

最终在`defineReactive`中，我们找到了如下的代码片段

```javascript
function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          dependArray(value)
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

分析了这么多，简单的表述就是以上`依赖收集`和`派发更新`的过程，这也就达到了更改数据后会触发其他订阅数据的联动。关于更多定于[数据watcher](http://www.zhangjinglin.cn/blog/d3cd78dbdf7ad7ae38d3bf38ded8ad95.html)请查看这篇文章

当然这是数据驱动的方式，那么双向数据绑定还要涉及到事件触发方法，方法处理数据。所以这就达到了view => Modal 的过程。

不过这里面我们经常看到的是关于`v-modal`的语法糖，我们一起来看[v-modal]()的处理流程。同时也借助这个语法糖从源码角度来看看事件的注册机制和解析方式。
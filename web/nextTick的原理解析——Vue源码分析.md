nextTick的原理解析——Vue源码分析
---

首先来看看以下Demo

```html
<template>
<div>
   <p ref="pmsg">{{msg}}</p>
   <button @click="output">click me</button>
</div>
</template>

<script>
    export default{
        name:'app',
        data(){
            return{
                msg : 'Just world!'
            }
        },
        methods:{
            output:function () {
                this.$nextTick(() => {
                    console.log('nextTick callback:', this.$refs.pmsg.innerText)
                })

                this.msg = 'Change !'
                console.log('now data:', this.$refs.pmsg.innerText)
                this.$nextTick().then(() => {
                    console.log('nextTick promise:', this.$refs.pmsg.innerText)
                })
            }
        }
    }
</script>
```

> 那么以上的输出结果是？

```text
$ nextTick callback:Just world!

$ now data:Just world!

$ nextTick promise:Change!

```

> 为什么为什么？迷糊一坨啊！

我们一起来看看官网怎么解释的

> 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

总之也是含糊化了，就是不晓得具体时间点和必须了解你的更新循环原理才能理解吧！

那我们一起深入了解一下Vue代码实现方式吧，其实很简单，代码在`src/core/util/env.js`

```javascript
export const nextTick = (function () {
  const callbacks = []
  let pending = false
  let timerFunc

  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  // the nextTick behavior leverages the microtask queue, which can be accessed
  // via either native Promise.then or MutationObserver.
  // MutationObserver has wider support, however it is seriously bugged in
  // UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
  // completely stops working after triggering a few times... so, if native
  // Promise is available, we will use it:
  /* istanbul ignore if */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
      p.then(nextTickHandler).catch(logError)
      // in problematic UIWebViews, Promise.then doesn't completely break, but
      // it can get stuck in a weird state where callbacks are pushed into the
      // microtask queue but the queue isn't being flushed, until the browser
      // needs to do some other work, e.g. handle a timer. Therefore we can
      // "force" the microtask queue to be flushed by adding an empty timer.
      if (isIOS) setTimeout(noop)
    }
  } else if (typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]'
  )) {
    // use MutationObserver where native Promise is not available,
    // e.g. PhantomJS IE11, iOS7, Android 4.4
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
      characterData: true
    })
    timerFunc = () => {
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    // fallback to setTimeout
    /* istanbul ignore next */
    timerFunc = () => {
      setTimeout(nextTickHandler, 0)
    }
  }

  return function queueNextTick (cb?: Function, ctx?: Object) {
    let _resolve
    callbacks.push(() => {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handleError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      timerFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        _resolve = resolve
      })
    }
  }
})()
```

还是个立即执行的函数，当然内部进行了是否支持`Promise`、`MutationObserver`和兜底方案`setTimeout`的集中`异步线程调用方式`支持测试，关于[异步线程调用方式](http://www.zhangjinglin.cn/blog/d37df4ef6d3bebce75e3cebbb62b65.html)请查阅相关原生Js文档

如果支持则采用其中一种方式去调用，后续`queueNextTick`就是将要执行的回调函数推到可执行的堆栈中，方便下个执行周期集中推出调用，切记这里是先进先出的方式调用`FIFO`

> 看吧很简单吧，实现原理，那到底什么时候调用`nextTick`呢？

我们一起查看一下吧`src/core/observer/scheduler.js`文件

```javascript
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true
      nextTick(flushSchedulerQueue)
    }
  }
}
```
看到了吧，就是等时机成熟的时候进行nextTick方法，输出所有相关watcher（user watcher、computed watcher、render watcher等）。

> 那什么时候时机成熟呢？

可以继续追溯到`src/core/observer/watcher.js`文件

```javascript
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }
```
哦，看到了就知道了当依赖收集周期已经完成后执行更新的时候开始，那依赖收集何时结束，请查看触发watche的`get`方法的具体代码。





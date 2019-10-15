promise的方案
---

### 有个相关的代码展示

更多promise的使用方案还是查阅MDN为准[promise in MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

这里主要展示一个DEMO：

```javascript
// 0.5秒后返回input*input的计算结果:
function multiply(input) {
    return new Promise(function (resolve, reject) {
        console.log('calculating ' + input + ' x ' + input + '...');
        setTimeout(resolve, 500, input * input);
    });
}

// 0.5秒后返回input+input的计算结果:
function add(input) {
    return new Promise(function (resolve, reject) {
        console.log('calculating ' + input + ' + ' + input + '...');
        setTimeout(resolve, 500, input + input);
    });
}

var p = new Promise(function (resolve, reject) {
    console.log('start new Promise...');
    resolve(123);
});

p.then(multiply)
 .then(add)
 .then(multiply)
 .then(add)
 .then(function (result) {
    log('Got value: ' + result);
});

```

最后的结果如下：

```text
start new Promise...

calculating 123 x 123...

calculating 15129 + 15129...

calculating 30258 x 30258...

calculating 915546564 + 915546564...

Got value: 1831093128
```

其实最主要的还是有关宏任务和微任务的具体使用场景，经常在各种业务逻辑中和面试中遇到

```javascript
setTimeout( () => {
  new Promise(resolve => {
    resolve()
    console.log(4)
  }).then(() => {
    console.log(7)
  })
})

new Promise(resolve => {
  resolve()
  console.log(1)
}).then( () => {
  console.log(3)
})

setTimeout( () => {
  Promise.resolve(6).then(() => console.log(6))
  new Promise(resolve => {
    resolve()
    console.log(8)
  }).then(() => {
    console.log(9)
  })
})

Promise.resolve(5).then(() => console.log(5))

console.log(2)

```

执行结果是什么呢?

> 1，2，3，5，4，7，8，6，9

为什么这样呢？我之前的博文中分析过这个流程，这里不再赘述，主要讲的就是要知道event loop的机制

![event loop](https://www.zhangjinglin.cn/images/u1.png)

![event loop](https://www.zhangjinglin.cn/images/u0.png)

以上便运行机制，可以站内搜索`promise`查看如何用原生实现promise的机制和方法






跟多相关博文请关注[promise腾讯云篇](https://cloud.tencent.com/developer/article/1476737)

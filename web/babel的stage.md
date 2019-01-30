babel的stage配置
---

stage成员如下：

+ stage-0 - Strawman: just an idea, possible Babel plugin.
+ stage-1 - Proposal: this is worth working on.
+ stage-2 - Draft: initial spec.
+ stage-3 - Candidate: complete spec and initial browser implementations.
+ stage-4 - Finished: will be added to the next yearly release.


> stage-0

stage-0 囊括了 1,2,3 的所有插件，另外再添加了

+ transform-do-expressions
+ transform-function-bind

这 2 个插件

transform-do-expressions 插件提供了 do 关键字，让你更方便的处理 if/else 语句，如下：

```javascript
let a = do {
  if(x > 10) {
    'big';
  } else {
    'small';
  }
};
// is equivalent to:
let a = x > 10 ? 'big' : 'small';
```
这个语法在 jsx 里使用特别有效, 以为我们不能这么写

```javascript
return (
            <div >
                {
                    if(name == 'a') { 
                        <AComponent/>; 
                    }else if(name == 'b') { 
                        <BComponent/>; 
                    }else { 
                        <CComponent/>; }
                    }
                }
            </div>
        )

```
但是有了 transform-do-expressions 插件 ， 我们可以这样写

```javascript
    return (
            <div >
               {do {
                    if(name == 'a') { 
                        <AComponent/>; 
                    }else if(name == 'b') { 
                        <BComponent/>; 
                    }else { 
                        <CComponent/>; }
                    }
                }}
            </div>
        )

```

然后看下 transform-function-bind 插件

它提供了 :: 操作符来简化上下文切换操作, 作为 bind , call的替代方案

```javascript
    obj::func
    // 等于
    func.bind(obj)
    
    obj::func(val)
    // 等于
    func.call(obj, val)
    
    ::obj.func(val)
    // 等于
    func.call(obj, val)
```

在react里，你甚至可以这样来绑定自己

```javascript
   <Components onClick={::this.change}>
```

或者用更高级的用法,像这样:

```javascript
    const { map, filter } = Array.prototype;
    
    let sslUrls = document.querySelectorAll('a')
                    ::map(node => node.href)
                    ::filter(href => href.substring(0, 5) === 'https');
    
    console.log(sslUrls);
```

题外话, 你也可以用修饰器的方式来做方法绑定,如使用 core-decorators 工具库的 @autobind

> stage-1

stage-1 则是囊括了 2 和 3 插件，另外增加了

+ transform-class-constructor-call (Deprecated)

+ transform-export-extensions

transform-export-extensions 作为export方法的扩展


```javascript
    export * as ns from 'mod';
    export v from 'mod';
```

transform-class-constructor-call 已不建议使用，这里不做介绍

> stage-2

同理，他拥有 3 的插件，还有

+ syntax-dynamic-import

+ transform-class-properties

+ syntax-dynamic-import 用于动态 import ，我们知道 import 语法只支持静态加载模块

import() 作为一个提案， 具体可以看这里 ，这个插件也只有被 babel内部使用，所以不再介绍, airbnb 提供了一个可用 插件 。

transform-class-properties 用与 class 的属性转化 ，如下

```javascript
class Bork {
    // 用于处理如下语法
    instanceProperty = "bork";
    boundFunction = () => {
      return this.instanceProperty;
    }

    // 静态属性
    static staticProperty = "babelIsCool";
    static staticFunction = function() {
      return Bork.staticProperty;
    }
  }

  let myBork = new Bork;

  // 可以看到 boundFunction 不是挂在原型上
  console.log(myBork.prototype.boundFunction); // > undefined

  // 被挂到了实例上
  console.log(myBork.boundFunction.call(undefined)); // > "bork"

  // 静态方法调用 （静态属性是挂在原型上的）
  console.log(Bork.staticFunction()); // > "babelIsCool"
```

> stage-3

拥有的插件

+ transform-object-rest-spread

+ transform-async-generator-functions

transform-object-rest-spread 用来处理 rest spread

```javascript
    let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
    console.log(x); // 1
    console.log(y); // 2
    console.log(z); // { a: 3, b: 4 }
```

```javascript
    let n = { x, y, ...z };
    console.log(n); // { x: 1, y: 2, a: 3, b: 4 }
```

transform-async-generator-functions 用来处理 async 和 await

```javascript
    async function* agf() {
      await 1;
      yield 2;
    }
```










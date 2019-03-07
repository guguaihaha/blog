Object.defineProperty
---

Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

### 语法

Object.defineProperty(obj, prop, descriptor)

> 参数

+ obj

要在其上定义属性的对象。
  
+ prop  

要定义或修改的属性的名称

+ descriptor

将被定义或修改的属性描述符

> 返回值

被传递给函数的对象

@注意：在ES6中，由于 Symbol类型的特殊性，用Symbol类型的值来做对象的key与常规的定义或修改不同，而Object.defineProperty 是定义key为Symbol的属性的方法之一。


### 描述

该方法允许精确添加或修改对象的属性。通过赋值操作添加的普通属性是可枚举的，能够在属性枚举期间呈现出来（for...in 或 Object.keys 方法）， 这些属性的值可以被改变，也可以被删除。这个方法允许修改默认的额外选项（或配置）。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的


### 属性描述符

对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。数据描述符是一个具有值的属性，该值可能是可写的，也可能不是可写的。存取描述符是由getter-setter函数对描述的属性。描述符必须是这两种形式之一；不能同时是两者

> 数据描述符和存取描述符均具有以下可选键值

+ configurable

 当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认为 false
 
+ enumerable

当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。默认为 false

> 数据描述符同时具有以下可选键值：

+ value

该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined

+ writable

当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为 false。

> 存取描述符同时具有以下可选键值

+ get

一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入this对象（由于继承关系，这里的this并不一定是定义该属性的对象）

默认为 undefined。

+ set

一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值

默认为 undefined


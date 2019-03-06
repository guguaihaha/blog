javascript知识点—reduce的使用方法和技巧
---

### 概念

reduce() 方法接收一个函数作为累加器（accumulator），数组中的每个值（从左到右）开始缩减，最终为一个值。

reduce 为数组中的每一个元素依次执行回调函数，不包括数组中被删除或从未被赋值的元素，接受四个参数：初始值（或者上一次回调函数的返回值），当前元素值，当前索引，调用 reduce 的数组。

### 语法

```javascript
 array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
```

> 参数解析

  + function(total, currentValue, currentIndex, arr)
  
  必需。用于执行每个数组元素的函数。
  
  内部参数解析
   
   - total   
      
      必须。初始值, 或者计算结束后的返回值。
      
   - currentValue
   
      必需。当前元素
      
   - currentIndex
    
     可选。当前元素的索引
     
   - arr
   
     可选。当前元素所属的数组对象。
     
   
   +  initialValue
   
   可选。传递给函数的初始值。一般填写为`0` 
   
   
     
          
### 简单应用

> demo 1:

```javascript
    var items = [10, 120, 1000];
    
    // our reducer function
    var reducer = function add(sumSoFar, item) { return sumSoFar + item; };
    
    // do the job
    var total = items.reduce(reducer, 0);
    
    console.log(total); // 1130
```

可以看出，reduce函数根据初始值0，不断的进行叠加，完成最简单的总和的实现。

reduce函数的返回结果类型和传入的初始值相同，上个实例中初始值为number类型，同理，初始值也可为object类型


> demo 2:

```javascript
    var items = [10, 120, 1000];
    
    // our reducer function
    var reducer = function add(sumSoFar, item) {
      sumSoFar.sum = sumSoFar.sum + item;
      return sumSoFar;
    };
    
    // do the job
    var total = items.reduce(reducer, {sum: 0});
    
    console.log(total); // {sum:1130}
```




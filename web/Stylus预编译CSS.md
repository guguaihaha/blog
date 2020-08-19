Stylus预编译CSS
---

大家比较熟悉的CSS预编译常用`Less`、`SASS/SCSS`，今天分享的是`Stylus`。

首先特性是啥？

富于表现力、动态的、健壮的 CSS

看不懂吗？那么就看看以下使用流程

+ 常用的CSS写法如下

```stylus
body {
  font: 12px Helvetica, Arial, sans-serif;
}
a.button {
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
}
```

+ 省略了花括号怎么样?

```stylus
body
  font: 12px Helvetica, Arial, sans-serif;

a.button
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
```

+ 再把分号省略掉呢？

```stylus
body
  font: 12px Helvetica, Arial, sans-serif

a.button
  -webkit-border-radius: 5px
  -moz-border-radius: 5px
  border-radius: 5px
```

+ 保持整洁

```stylus
border-radius()
  -webkit-border-radius: arguments
  -moz-border-radius: arguments
  border-radius: arguments

body
  font: 12px Helvetica, Arial, sans-serif

a.button
  border-radius(5px)
```

+ 让混合（mixin）变得透明怎么样？

```stylus
border-radius()
  -webkit-border-radius: arguments
  -moz-border-radius: arguments
  border-radius: arguments

body
  font: 12px Helvetica, Arial, sans-serif

a.button
  border-radius: 5px
```

+ 创建与分享

```stylus
@import 'vendor'

body
  font: 12px Helvetica, Arial, sans-serif

a.button
  border-radius: 5px
```


+ 甚至是语法内函数！

```stylus
sum(nums...)
  sum = 0
  sum += n for n in nums

sum(1 2 3 4)
// => 10
```

+ 如果所有这些都是可选的又将怎样？

```stylus
fonts = Helvetica, Arial, sans-serif

body {
  padding: 50px;
  font: 14px/1.4 fonts;
}
```

> 那么关于Stylus的特性有哪些？

+ 冒号可有可无
+ 分号可有可无
+ 逗号可有可无
+ 括号可有可无
+ 变量
+ 插值（Interpolation）
+ 混合（Mixin）
+ 数学计算
+ 强制类型转换
+ 动态引入
+ 条件表达式
+ 迭代
+ 嵌套选择器
+ 父级引用
+ Variable function calls
+ 词法作用域
+ 内置函数（超过 60 个）
+ 语法内函数（In-language functions）
+ 压缩可选
+ 图像内联可选
+ Stylus 可执行程序
+ 健壮的错误报告
+ 单行和多行注释
+ CSS 字面量
+ 字符转义
+ TextMate 捆绑
+ 以及更多！[官网](https://stylus.bootcss.com/docs/selectors.html)

以上基本表述了常见的使用方法，贴一段示例代码

```stylus
$background-color = lightblue
add (a, b = a)
    a = unit(a, px)
    b = unit(b, px)
    a + b

.list-item
.text-box
    span
        background-color: $background-color
        margin: add(10)
        padding: add(10, 5)
    &:hover
        background-color: powderblue
```

编译后

```stylus
.list-item span,
.text-box span {
  background-color: #add8e6;
  margin: 20px;
  padding: 15px
}
.list-item:hover,
.text-box:hover {
  background-color: #b0e0e6;
}
```
感觉很有意思就尝试试试吧。

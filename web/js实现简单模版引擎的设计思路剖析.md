js实现简单模版引擎设计思路剖析
---

本文分享一下关于如何快速实现js模版引擎的替换知识

#### 首先需要注意本文的重点

+ 只剖析如何执行的思路，不强调性能

+ 简单的模型关系，复杂的操作可以自行拼凑

+ 涵盖的思路包含目前大多数框架都使用的方式和方法

#### 准备的知识点

+ 正则表达式

+ new Function

+ with(this){}

+ js的基本知识

+ es6模版字符串替换思路

好了，那么就直接剖析使用思路

```javascript
(new Function("var a = [];a.push(1);a.push(2);return a.join(',');"))()
```
可以看到最后可以正确执行，这就是本文的重点。如果涉及到具体对象的时候，我们可以通过传参

```javascript
var data = {
    title : "test demo",
    subTitle : "other demo"
}
(new Function("_data","var a = [];a.push(_data.title);a.push(_data.subTitle);return a.join(',');"))(data)
```
就这样可以使用了哦，如果模式比较复杂，简化代码的话可以参考结合使用 `with` 对象一起使用

好了，如果是`for 或者 if 等`稍微复杂的逻辑运算怎么办呢？

其实这也简单，就是利用js的代码执行即可。下面我们来一起找个DEMO

#### 案例分析

```javascript
<script type="text/x-nv" id="nv-demo">
    <h3>{%=o.title%}</h3>
    <p>Released under the
    <a href="{%=o.license.url%}">{%=o.license.name%}</a></p>
    <h4>Features</h4>
    <ul>
        {% for (var i=0; i<o.features.length; i++) { %}
        <li>{%=o.features[i]%}</li>
        {% } %}
    </ul>
</script>
var o = {
        "title": "JavaScript Templates",
        "license": {
            "name": "MIT license",
            "url": "https://opensource.org/licenses/MIT"
        },
        "features": [
            "lightweight & fast",
            "powerful",
            "zero dependencies"
            ]
        }
```

看上面的demo，我们要把`o`对象作为数据，把`nv-demo`中的代码作为模版，最后输出正确结果

我们直接把最后嵌套的结果输出出来

```javascript
var jspe = '\n    <h3>' + o.title + '</h3>\n    <p>Released under the\n    <a href="' + o.title + '">' + o.title + '</a></p>\n    <h4>Features</h4>\n    <ul>\n        ';
for (var i = 0; i < o.features.length; i++) {
    jspe += '\n        <li>' + o.title + '</li>\n        ';
}
jspe += '\n    </ul>\n';
return jspe
```
把以上结果放到`new Function`中运行即可

那么我们是如何去做呢，首先我们需要运用到正则进行全局匹配，为了方便讲解我们拆分正则进行分析

在分析之前，我们约定一下规则

+ 模版对象都是以`o`来进行对象传递的

+ {%=o.xxx%} 这样的状态是输出文本，如果是html请转移，自行配置正则即可

+ {% for or if %} 切记必须在`{% `添加空格和` %}`添加空格，否则无法识别

只要知道以上三个点即可

```javascript
var StringText = document.getElementById('nv-demo').innerText,
    textString = "var jspe = '"
    textString += StringText.replace(/\s/g,function (s) {
        return {
            '\n': '\\n',
            '\r': '\\r',
            '\t': '\\t',
            ' ': ' '
        }[s] || '\\' + s
    }).replace(/\{\%\=([\o\.\w|\.|\[|\]]+)\%\}/g,"'+ $1 +'")
        .replace(/\{\%\s/g,"'; ")
        .replace(/\s\%\}/g," jspe+='")
    textString +="';return jspe"
    //执行
    new Function('o',textString)(o)
```
代码思路分析

+ 首先获取`nv-demo`的内容，然后拼接字符串

+ 第一个 `replace` 是进行空格，换行等字符的正确替换转义

+ 第二个 `replace` 是针对`{%=o.xxx%}`的字符进行替换拼接对象 `'+ o.xxx +'`

+ 第三个 `replace` 是针对`{% `的逻辑运算进行替换结束对象 `'; `

+ 第四个 `replace` 是针对 ` %}`的起始拼接 `jspe+='`

+ 最后结束并返回结果即可

有人问，为什么这么做，起始你就把这个字符串作为js引擎可执行的对象来进行严格拼接即可。
逻辑运算分割开来，最后把string对象返回就是最后输出结果

当然如果还想有其他的筛选等自定义行为都可以在这个过程中进行替换，以上进行了多次搜索匹配，性能是很低的

给个一次性匹配但结果需要多次分析的正则` /([\s'\\])(?!(?:[^{]|\{(?!%))*%\})|(?:\{%(=|#)([\s\S]+?)%\})|(\{%)|(%\})/g`。具体完整代码片段请

查阅[完整的正则替换](http://www.zhangjinglin.cn/blog/d3dd3cf7def5ef8e77eb6d39d6d8ad95.html)

好了，喜欢的给点个赞吧，谢谢



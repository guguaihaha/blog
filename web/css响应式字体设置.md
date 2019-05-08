CSS响应式字体如何设置
---

在进行页面响应式设计中，往往需要根据屏幕分辨率来显示不同大小的字体。
通常的做法是通过media queries给不同的分辨率指定不同的字体样式

> 通常做法

```css
body
{
       font-size: 22px; 
}
h1
{
       font-size:44px;
}

@media (min-width: 768)
{
       body
       {
           font-size: 17px; 
       }
       h1
       {
           font-size:24px;
       }
}
```

除此之外，我们还可以通过下面的方式让字体自适应屏幕分辨率。

> 建议用法
  
```text
1vw = viewport宽度的1%
1vh = viewport高度的1%
1vmin = 1vw或者1vh中较小的值
1vmax = 1vw或者1vh中较大的值
```  

如下：

```css

h1 {
  font-size: 5.9vw;
}

h2 {
  font-size: 3.0vh;
}

p {
  font-size: 2vmin;
}
```

**除此之外，普及一下相关知识点：**

> 什么是viewport?

viewport是HTML5中新加入的一个meta标记，其主要作用是为移动客户端的浏览器进行显示优化。通过设置viewport的属性值，可以控制当前页面默认采用什么样的方式在移动端的浏览器中显示页面。下面是一个常用的针对移动网页优化过的页面的viewport meta标记的设置项：

```html
<meta name="viewport" content="width =device-width, initial-scale=1, maximum-scale=1"/>
```
如果想让页面支持响应式设计，需要给页面添加viewport meta标记。[Bootstrap中的响应式设计](https://v2.bootcss.com/scaffolding.html#responsive)

完整的viewport语法如下：

```html
<meta name="viewport"
        content="
            height = [pixel_value | device-height] ,
            width = [pixel_value | device-width ] ,
            initial-scale = float_value ,
            minimum-scale = float_value ,
            maximum-scale = float_value ,
            user-scalable = [yes | no] ,
            target-densitydpi = [dpi_value | device-dpi | high-dpi | medium-dpi | low-dpi]
        "
/>
```
### 含义解释：

+ height：控制viewport的高度，可以指定一个固定的值，或者device-height来表示设备的高度（单位为缩放100%时的像素值）。

+ width：和height对应，表示viewport的宽度。devive-width表示设备的高度。

+ initial-scale：页面的初始缩放比例，值允许为小数，表示当前页面大小的倍数。例如2.0表示页面初始状态下会被放大2倍。

+ minimum-scale：最小允许缩放比例，值允许为小数，表示页面最小能以多大的倍数显示。例如2.0表示页面不能缩小到2倍以下进行显示。

+ maxmium-scale：和minimun-scale对应，表示最大允许缩放比例。

+ user-scalable：是否允许用户缩放页面。默认值为yes，当设置为no时minimum-scale和maximum-scale无效。

+ target-densitydpi：指定页面在什么样的dpi下显示。屏幕像素密度是由屏幕分辨率来决定的，通常定义为每英寸点的数量，即dpi。Android支持三种dpi设置：低像素密度（low-dpi），中像素密度（medium-dpi），高像素密度（high-dpi）。低像素密度的屏幕每英寸上的像素点少，而高像素密度的屏幕每英寸上的像素点多。Android Browser和WebView默认屏幕为中像素密度。也可以直接指定一个具体的dpi值，该值允许的范围为70-400之间。device-dpi表示以设备默认的dpi来显示页面。

**注意：所有的缩放值都必须在0.01-10的范围之内，否则无效。**

### CSS中几种不同单位之间的比较

+ px：像素（Pixel）。相对长度单位，所占大小由屏幕分辨率决定。

+ em：相对长度单位。相当于当前对象内文本的字体尺寸，如果当前对行内文本的字体尺寸未被认为设置，则相对于浏览器的默认字体尺寸。em的值并不是固定的，它会继承父级元素的字体大小。所有未经调整的浏览器都符合: 1em=16px。那么12px=0.75em,10px=0.625em。为了简化font-size的换算，需要在css中的body选择器中声明Font-size=62.5%，这就使em值变为 16px*62.5%=10px, 这样12px=1.2em, 10px=1em, 也就是说只需要将你的原来的px数值除以10，然后换上em作为单位就行了。

+ rem：CSS3新增的一个相对单位。与em的主要区别在于使用rem为元素设定字体大小时，仍然是相对大小，但相对的只是HTML根元素。这个单位可谓集相对大小和绝对大小的优点于一身，通过它既可以做到只修改根元素就成比例地调整所有字体大小，又可以避免字体大小逐层复合的连锁反应。目前，除了IE8及更早版本外，所有浏览器均已支持rem。对于不支持它的浏览器，应对方法也很简单，就是多写一个绝对单位的声明。这些浏览器会忽略用rem设定的字体大小。

+ pt：印刷业上常使用的单位，一般用于页面打印排版，即磅的意思。

+ %：另外我们还可以使用百分比来指定大小，它表示当前字体相对于浏览器默认字体大小的倍数。该单位在页面响应式设计中也被经常用到。

+ vw/vh/vmin/vmax：上面已经介绍了，表示字体相对于viewport高或宽的大小。

  


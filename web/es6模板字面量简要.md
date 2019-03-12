es6模版模板字面量
---

上来还是一样，先来个demo

### 基本用法：

模板字面量是增强版的字符串，它用反引号（`）标识

```javascript
let message = `Hello world!`;
console.log(message); // "Hello world!"
console.log(typeof message); // "string"
console.log(message.length); // 12
```

以上就是普通的赋值操作，就是个普通的字符串

如果想要引入特殊符号，包含` \` `的话请使用` \ `来转义即可

```javascript
let message = `\`Hello\` world!`;
console.log(message); // "`Hello` world!"
console.log(typeof message); // "string"
console.log(message.length); // 14
```

### 多行字符串

```javascript
let message = `Multiline
string`;
// "Multiline
// string"
console.log(message); 
console.log(message.length); // 16
```

切记，反引号之内空格都是会被记录的，所以要格外注意

以下操作小技巧也可以使用：

trim:

```javascript
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

`\n`的指定换行插入符号也会被识别的

```javascript
let message = `Multiline\nstring`;
// "Multiline
// string" 
console.log(message); 
console.log(message.length); // 16
```

### 变量占位符

模板字面量看上去仅仅是普通JS字符串的升级版，但二者之间真正的区别在于模板字面量的变量占位符。变量占位符允许将任何有效的JS表达式嵌入到模板字面量中，并将其结果输出为字符串的一部分

变量占位符由起始的 ${ 与结束的 } 来界定，之间允许放入任意的 JS 表达式。最简单的变量占位符允许将本地变量直接嵌入到结果字符串中

```javascript
let name = "Nicholas",
message = `Hello, ${name}.`;
console.log(message); // "Hello, Nicholas."
```
占位符 ${name} 会访问本地变量 name ，并将其值插入到 message 字符串中。 message变量会立即保留该占位符的结果

既然占位符是JS表达式，那么可替换的就不仅仅是简单的变量名。可以轻易嵌入运算符、函数调用等

```javascript
let count = 10,
price = 0.25,
message = `${count} items cost $${(count * price).toFixed(2)}.`;
console.log(message); // "10 items cost $2.50."
```

```javascript
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```

模板字面量本身也是 JS 表达式，因此可以将模板字面量嵌入到另一个模板字面量内部

```javascript
let name = "Nicholas",
    message = `Hello, ${
        `my name is ${ name }`
    }.`;
console.log(message); // "Hello, my name is Nicholas."
```

### 模版标签

模板字面量真正的威力来自于标签模板，每个模板标签都可以执行模板字面量上的转换并返回最终的字符串值。标签指的是在模板字面量第一个反引号'`'前方标注的字符串

```javascript
let message = tag`Hello world`;
```

在这个示例中， tag 就是应用到 `Hello world` 模板字面量上的模板标签

> 定义标签

标签可以是一个函数，调用时传入加工过的模板字面量各部分数据，但必须结合每个部分来创建结果。第一个参数是一个数组，包含Javascript解释过后的字面量字符串，它之后的所有参数都是每一个占位符的解释值

标签函数通常使用不定参数特性来定义占位符，从而简化数据处理的过程

```javascript
function tag(literals, ...substitutions) {
  // 返回一个字符串
}
```

为了进一步理解传递给tag函数的参数，查看以下代码

```javascript
let count = 10,
price = 0.25,
message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
```
如果有一个名为passthru()的函数，那么作为一个模板字面量标签，它会接受3个参数首先是一个literals数组，包含以下元素

+ 第一个占位符前的空字符串("")

+ 第一、二个占位符之间的字符串(" items cost $")

+ 第二个占位符后的字符串(".")

下一个参数是变量count的解释值，传参为10，它也成为了substitutions数组里的第一个元素

最后一个参数是(count*price).toFixed(2)的解释值，传参为2.50，它是substitutions数组里的第二个元素

**literals里的第一个元素是一个空字符串，这确保了literals[0]总是字符串的始端，就像literals[literals.length-1]总是字符串的结尾一样。substitutions的数量总比literals少一个，这也意味着表达式substitutions. Iength === literals. Iength-1的结果总为true**

```javascript
var a = 5;
var b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```
通过这种模式，我们可以将literals和substitutions两个数组交织在一起重组结果字符串。先取出literals中的首个元素，再取出substitution中的首个元素，然后交替继续取出每一个元素，直到字符串拼接完成。于是可以通过从两个数组中交替取值的方式模拟模板字面量的默认行为

```javascript
function passthru(literals, ...substitutions) {
    let result = "";
    // 仅使用 substitution 的元素数量来进行循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }
    // 添加最后一个字面量
    result += literals[literals.length - 1];
    return result;
}
let count = 10,
price = 0.25,
message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
console.log(message); // "10 items cost $2.50."
```
这个示例定义了一个passthru标签，模拟模板字面量的默认行为，展示了一次转换过程。此处的小窍门是使用substitutions.length来为循环计数

> Another demo

“标签模板”的一个重要应用，就是过滤HTML字符串，防止用户输入恶意内容

```javascript
var message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

function SaferHTML(templateData) {
  var s = templateData[0];
  for (var i = 1; i < arguments.length; i++) {
    var arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
```
上面代码中，sender变量往往是用户提供的，经过SaferHTML函数处理，里面的特殊字符都会被转义


```javascript
var sender = '<script>alert("abc")</script>'; // 恶意代码
var message = SaferHTML`<p>${sender} has sent you a message.</p>`;

console.log(message);// <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>
```

标签模板的另一个应用，就是多语言转换（国际化处理）

```javascript
i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
// "欢迎访问xxx，您是第xxxx位访问者！"
```

模板字符串本身并不能取代模板引擎，因为没有条件判断和循环处理功能，但是通过标签函数，可以自己添加这些功能

```javascript
// 下面的hashTemplate函数
// 是一个自定义的模板处理函数
var libraryHtml = hashTemplate`
  <ul>
    #for book in ${myBooks}
      <li><i>#{book.title}</i> by #{book.author}</li>
    #end
  </ul>
`;
```

### raw()

String.raw方法，往往用来充当模板字面量的处理函数，返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，对应于替换变量后的模板字面量

```javascript
let message1 = `Multiline\nstring`,
message2 = String.raw`Multiline\nstring`;
console.log(message1); // "Multiline
// string"
console.log(message2); // "Multiline\\nstring"
```

```javascript
String.raw`Hi\n${2+3}!`;
// "Hi\\n5!"

String.raw`Hi\u000A!`;
// 'Hi\\u000A!'
```

如果原字符串的斜杠已经转义，那么String.raw不会做任何处理

```javascript
String.raw`Hi\\n`// "Hi\\n"
```
String.raw方法可以作为处理模板字面量的基本方法，它会将所有变量替换，而且对斜杠进行转义，方便下一步作为字符串来使用。

String.raw方法也可以作为正常的函数使用。这时，它的第一个参数，应该是一个具有raw属性的对象，且raw属性的值应该是一个数组

```javascript
String.raw({ raw: 'test' }, 0, 1, 2);// 't0e1s2t'

// 等同于
String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);
```
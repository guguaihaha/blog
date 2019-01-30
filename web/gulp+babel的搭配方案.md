 gulp + babel的搭配方案
---
首先你要新建项目，然后安装项目需要的依赖，这篇主要讲述如何结合babel的注意事项


当然第一步还是安装，推荐用[yarn](https://yarnpkg.com/zh-Hans/)进行安装

```text
yarn add gulp-babel --dev
```

然后就是创建任务代码片段了

```javascript
     var gulp = require("gulp");
     var sourcemaps = require("gulp-sourcemaps");
     var babel = require("gulp-babel");
     var concat = require("gulp-concat");
     
     gulp.task("default", function () {
       return gulp.src("src/**/*.js")
         .pipe(sourcemaps.init())
         .pipe(babel())
         .pipe(concat("all.js"))
         .pipe(sourcemaps.write("."))
         .pipe(gulp.dest("dist"));
     });
```

到这里就可以运行了吗？新版本的gulp-babel的确生效了，不过建议还是要配置[.babelrc](https://www.babeljs.cn/docs/usage/babelrc)的文件并开启[插件]()
最新版本的babel都会集中安装大部分常用依赖在 `node_modules/@babel` 目录下,以下为`.babelrc`的配置

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules":false,
        "targets": {
          "browsers":["ie >= 7"]
        },
        "useBuiltIns":"usage",
        "debug":true,
        "loose":true
      }
    ]
  ],
  "plugins":[
    "@babel/plugin-transform-runtime",
    "@babel/plugin-transform-modules-umd"
  ]
}
```

---
### 解释：

> 以上代码中`loose`为Babel的插件模式

+ 尽可能符合ECMASCRIPT6语义的NORMAL模式，默认是此项

+ 提供更简单ES5代码的LOOSE模式，推荐此项

通常，推荐不使用loose模式，使用这种模式的优点和缺点是：

+ 优点：生成的代码可能更快，对老的引擎有更好的兼容性，代码通常更简洁，更加的“ES5化”

+ 缺点：你是在冒险——随后从转译的ES6到原生的ES6时你会遇到问题。这个险是很不值得冒的

> 以上代码中`useBuiltIns` 

+ 如果useBuiltIns为true，项目中必须引入babel-polyfill，同时比直接入口import/require打包后的体积小很多，因为此模式是按需引入

+ 可选值包括："usage" | "entry" | false, 默认为 false，表示不对 polyfills 处理，这个配置是引入 polyfills 的关键。

+ "useBuiltIns":"usage"，在文件需要的位置单独按需引入，可以保证在每个bundler中只引入一份。

+ "useBuiltIns":"entry"， 在项目入口引入一次

+ "useBuiltIns": false，不在代码中使用polyfills，表现形式和@babel/preset-latest一样，当使用ES6+语法及API时，在不支持的环境下会报错。

> 以上代码中的`@babel/preset-env`

替换之前所有babel-presets-es20xx插件，也就是说，这是一个能根据运行环境为代码做相应的编译，@babel/preset-env的推出是为了解解决个性化输出目标代码的问题，通过browserslist语法解析需要支持的目标环境，根据环境将源代码转义到目标代码，可以实现代码精准适配。
                             

此外，`@babel/preset-env`不包含`state-x一些列插件`，只支持最新推出版本的JavaScript语法（state-4），关于`state-x`后面会介绍。

替换@babel/plugin-transform-runtime的使用

`@babel/plugin-transform-runtime`插件是为了解决：

+ 多个文件重复引用相同helpers（帮助函数）-> 提取运行时

+ 新API方法全局污染 -> 局部引入

这个插件推荐在编写library/module时使用。当然，以上问题可通过设置useBuiltIns搞定。

###使用说明

默认情况下，@babel/preset-env的效果和@babel/preset-latest一样，虽然上面的说明有提到polyfills，但是也需要在配置中设置useBuiltIns才会生效。


> 以上代码中的`targets`

设置支持环境，支持的key包括：chrome, opera, edge, firefox, safari, ie, ios, android, node, electron。

比如：

```json
{
  "presets": [
    ["@babel/env", {
      "targets": {
        "node": "current",
        "chrome": 52,
        "browsers": ["last 2 versions", "safari 7"]
      }
    }]
  ]
}
```
其中，`browserslist`可在`package.json`中配置，这个和设置CSS的`Autoprefixer`一致，配置优先级如下：

```text
targets.browsers > package.json/browserslist
```

此外，`browserslist`的配置满足最大匹配原则，比如需要同时支持在 IE 8 和 Chrome 55 下运行，则`preset-env`提供所有 IE 8 下需要的插件，即使 Chrome 55 可能不需要。


> 以上代码中的`modules`


选项用于模块转化规则设置，可选配置包括："amd" | "umd" | "systemjs" | "commonjs" | false, 默认使用 "commonjs"。即，将代码中的ES6的import转为require。
如果你当前的webpack构建环境是2.x/3.x，推荐将modules设置为false。具体原因就是模块化注入和模块化打包的软件`webpack`等冲突


#### 常见问题：

+ 这个和@babel/preset-env的区别

`@babel/preset-env`会根据预设的浏览器兼容列表从`stage-4`选取必须的plugin，也就是说，不引入别的`stage-x`，`@babel/preset-env`将只支持到stage-4

#### 建议

1. 如果是React用户，建议配到`@babel/preset-stage-0`

其中的两个插件对于写JSX很有帮助。

+ transform-do-expressions：if/else三目运算展开
+ transform-function-bind：this绑定

2. 通常使用建议配到`@babel/preset-stage-2`

插件包括：

+ syntax-dynamic-import： 动态import
+ transform-class-properties：用于 class 的属性转化
+ transform-object-rest-spread：用来处理 rest spread
+ transform-async-generator-functions：用来处理 async 和 await

#### @babel/polyfill

这个插件是对core-js和regenerator-runtime的再次封装，在`@babel/preset-env`中的`useBuiltIns: entry`用到，代码不多。

```javascript
if (global._babelPolyfill) {
  throw new Error("only one instance of @babel/polyfill is allowed");
}
global._babelPolyfill = true;

import "core-js/shim";
import "regenerator-runtime/runtime";
```
##### core-js/shim

只包含了纳入标准的API及实例化方法，例如下列常见的。注意，这里没有generator/async，如果需要，那就安装插件`@babel/preset-stage-3`（或者0 / 1 / 2）

```javascript
require('./modules/es6.string.trim');
require('./modules/es6.string.includes');
require('./modules/es7.array.includes');
require('./modules/es6.promise');
```

主要是给generator/async做支持的插件。


> At the ending

这里需要理解下三个概念：

+ 最新ES 语法：比如，箭头函数
+ 最新ES API：，比如，Promise
+ 最新ES 实例方法：比如，String.protorype.includes

`@babel/preset-env`默认支持语法转化，需要开启`useBuiltIns`配置才能转化API和实例方法。
此外，不管是写项目还是写Library/Module，使用`@babel/preset-env`并正确配置就行。多看英文原稿说明，中文总结看看就好，别太当真。




---



基于以上配置，可以说就能使用babel了，如果内嵌了import和require的模块化加载方案，可以结合打包工具webpack、rollup、browserify等
这里主要说一下browserify的结合使用方案。

```text
yarn add gulp-browserify
```
然后引入到入口文件

```javascript
   var browserify       = require('gulp-browserify');
   var gulp = require("gulp");
   var sourcemaps = require("gulp-sourcemaps");
   var babel = require("gulp-babel");
   var concat = require("gulp-concat");
  
  var gulp = require("gulp");
       var sourcemaps = require("gulp-sourcemaps");
       var babel = require("gulp-babel");
       var concat = require("gulp-concat");
       
       gulp.task("default", function () {
         return gulp.src("src/**/*.js")
           .pipe(sourcemaps.init())
           .pipe(babel())
           .pipe(browserify({
              insertGlobals : true,
           }))
           .pipe(concat("all.js"))
           .pipe(sourcemaps.write("."))
           .pipe(gulp.dest("dist"));
       });
```
关于[gulp-browserify](https://github.com/deepak1556/gulp-browserify)查看语法API即可，
到此就可以进行babel转换和打包同时进行，要注意browserify需要使用commonJs格式，如果使用import的方式引入，可以通过.babelrc中添加插件
`@babel/plugin-transform-modules-commonjs`来解决，至此全部搞定。

如果有相关问题请一起[讨论](https://github.com/guguaihaha/blog/issues/2)


portfinder帮你自动获取可用端口
---

今天分享的内容很简单，就是在使用各种基于nodeJs环境下开发的server服务下，如何获取可用端口？

往往端口都固定并且被占用，如果想设计一个自动获取的端口工具函数，一般可能会这么写

```typescript
const net = require('net');

var server = net.createServer(() => { })

function nextPort(port: number): number{
    // 端口自动+1
    return port + 1;
}

function getPort(port: number, callback){
    function onListen(data: object) {
      // 链接成功，关闭端口并注销事件
      callback(null, port);
      server.removeListener("error", onError);
      server.close();
    }
    
    function onError(err) {
      // 链接失败，端口重新尝试+1
      getPort(nextPort(port));
    }
    server.once('error', onError);
    server.once('listening', onListen);
    server.listen(port);
}

getPort(1024, (err,port) => {
    console.log(port);
})
```

是不是很熟悉的一种模式，就是通过`net`的中间件去进行尝试链接，直到找到不存在的端口号并能注册使用为止。

当然以上只是简单的例子，还有很多要考虑，比如端口区间设置等功能，今天推荐一个有意思的中间件`portfinder`

使用方式很简单：

```typescript
  var portfinder = require('portfinder');

  portfinder.getPort(function (err, port) {
    //
    // `port` 说明可以在此作用域使用
    //
  });
```
也可以使用promise方式

```typescript
  const portfinder = require('portfinder');

  portfinder.getPortPromise()
    .then((port) => {
        //
        // `port` 说明可以在此作用域使用
        //
    })
    .catch((err) => {
        //
        // 不能使用，肯定会返回具体错误信息`err`,比如`Error: listen EACCES: permission denied 127.0.0.1:80`等信息
        //
    });
```

另外还有其他配置属性可以使用，以下激活了全局的基本设置

```typescript
portfinder.basePort = 3000;    // 默认值: 8000
portfinder.highestPort = 3333; // 默认值: 65535
```

也支持局部定义

```typescript
portfinder.getPort({
    port: 3000,    // 最小端口号
    stopPort: 3333 // 最大端口号
}, callback);
```


具体文档地址[了解node-portfinder](https://github.com/http-party/node-portfinder)



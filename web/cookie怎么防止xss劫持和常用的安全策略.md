cookie怎么防止xss劫持
---

cookie是老生常谈的问题，估计很多人都是知道的如何使用cookie，我们也来一起简单介绍一下

cookie常用功能

#### 语法

var docCookies = document.cookie

## 写入cookie

```javascript
 docCookies.setItem(name, value[, end[, path[, domain[, secure]]]])
```
#### 参数
 
 **name (必要)**
 
 要创建或覆盖的cookie的名字 (string)。
 
 **value (必要)**
 
 cookie的值 (string)。
 
 **end (可选)**
 
 最大年龄的秒数 (一年为31536e3， 永不过期的cookie为Infinity) ，或者过期时间的GMTString格式或Date对象; 如果没有定义则会在会话结束时过期 (number – 有限的或 Infinity – string, Date object or null)。
 
 **path (可选)**
 
 例如 '/', '/mydir'。 如果没有定义，默认为当前文档位置的路径。(string or null)。路径必须为绝对路径
 
 **domain (可选)**
 
 例如 'example.com'， '.example.com' (包括所有子域名), 'subdomain.example.com'。如果没有定义，默认为当前文档位置的路径的域名部分 (string或null)。
 
 **secure (可选)**
 
 cookie只会被https传输 (boolean或null)。
 
 ## 得到cookie
 
```javascript
 docCookies.getItem(name)
```
 
 ## 移除cookie
 
```javascript
 docCookies.removeItem(name[, path],domain)
```
 
 ## 检测cookie
  
```javascript
 docCookies.hasItem(name)
```
  
 ## 得到所有 cookie 列表
    
```javascript
 docCookies.keys()
```

好了，找了一个官方推荐的cookie封装

```javascript
/*\
|*|
|*|  :: cookies.js ::
|*|
|*|  A complete cookies reader/writer framework with full unicode support.
|*|
|*|  https://developer.mozilla.org/en-US/docs/DOM/document.cookie
|*|
|*|  This framework is released under the GNU Public License, version 3 or later.
|*|  http://www.gnu.org/licenses/gpl-3.0-standalone.html
|*|
|*|  Syntaxes:
|*|
|*|  * docCookies.setItem(name, value[, end[, path[, domain[, secure]]]])
|*|  * docCookies.getItem(name)
|*|  * docCookies.removeItem(name[, path], domain)
|*|  * docCookies.hasItem(name)
|*|  * docCookies.keys()
|*|
\*/

var docCookies = {
  getItem: function (sKey) {
    return decodeURIComponent(document.cookie.replace(new RegExp("(?:(?:^|.*;)\\s*" + encodeURIComponent(sKey).replace(/[-.+*]/g, "\\$&") + "\\s*\\=\\s*([^;]*).*$)|^.*$"), "$1")) || null;
  },
  setItem: function (sKey, sValue, vEnd, sPath, sDomain, bSecure) {
    if (!sKey || /^(?:expires|max\-age|path|domain|secure)$/i.test(sKey)) { return false; }
    var sExpires = "";
    if (vEnd) {
      switch (vEnd.constructor) {
        case Number:
          sExpires = vEnd === Infinity ? "; expires=Fri, 31 Dec 9999 23:59:59 GMT" : "; max-age=" + vEnd;
          break;
        case String:
          sExpires = "; expires=" + vEnd;
          break;
        case Date:
          sExpires = "; expires=" + vEnd.toUTCString();
          break;
      }
    }
    document.cookie = encodeURIComponent(sKey) + "=" + encodeURIComponent(sValue) + sExpires + (sDomain ? "; domain=" + sDomain : "") + (sPath ? "; path=" + sPath : "") + (bSecure ? "; secure" : "");
    return true;
  },
  removeItem: function (sKey, sPath, sDomain) {
    if (!sKey || !this.hasItem(sKey)) { return false; }
    document.cookie = encodeURIComponent(sKey) + "=; expires=Thu, 01 Jan 1970 00:00:00 GMT" + ( sDomain ? "; domain=" + sDomain : "") + ( sPath ? "; path=" + sPath : "");
    return true;
  },
  hasItem: function (sKey) {
    return (new RegExp("(?:^|;\\s*)" + encodeURIComponent(sKey).replace(/[-.+*]/g, "\\$&") + "\\s*\\=")).test(document.cookie);
  },
  keys: /* optional method: you can safely remove it! */ function () {
    var aKeys = document.cookie.replace(/((?:^|\s*;)[^\=]+)(?=;|$)|^\s*|\s*(?:\=[^;]*)?(?:\1|$)/g, "").split(/\s*(?:\=[^;]*)?;\s*/);
    for (var nIdx = 0; nIdx < aKeys.length; nIdx++) { aKeys[nIdx] = decodeURIComponent(aKeys[nIdx]); }
    return aKeys;
  }
};
```

## 预防XSS的cookie


将cookie设置成HttpOnly是为了防止XSS攻击，窃取cookie内容，这样就增加了cookie的安全性，即便是这样，也不要将重要信息存入cookie。

如何在Java中设置cookie是HttpOnly呢看
Servlet 2.5 API 不支持 cookie设置HttpOnly

建议升级Tomcat7.0，它已经实现了Servlet3.0

但是苦逼的是现实是，老板是不会让你升级的。
那就介绍另外一种办法：
利用HttpResponse的addHeader方法，设置Set-Cookie的值
```javascript
cookie字符串的格式：key=value; Expires=date; Path=path; Domain=domain; Secure; HttpOnly
```

//设置cookie
```javascript
response.addHeader("Set-Cookie", "uid=112; Path=/; HttpOnly");
```

//设置多个cookie
```javascript
response.addHeader("Set-Cookie", "uid=112; Path=/; HttpOnly");
response.addHeader("Set-Cookie", "timeout=30; Path=/test; HttpOnly");
```

//设置https的cookie
```javascript
response.addHeader("Set-Cookie", "uid=112; Path=/; Secure; HttpOnly");
```

在实际使用中，我们可以使FireCookie查看我们设置的Cookie 是否是HttpOnly

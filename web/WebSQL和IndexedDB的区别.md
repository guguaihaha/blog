WebSQL和IndexedDB的区别
---

> WebSQL

WebSQL目前只有谷歌支持较好，如图：

![兼容图示](https://www.zhangjinglin.cn/blogimg/w1.png)

**以下部分摘抄网络用例和解释**

我们对数据库的一般概念是后端才会跟数据库打交道，进行一些业务性的增删改查。而这里的数据库也不同于真正意义上的数据库。

废话少说，先出招吧：

主要方法：

+ openDatabase：这个方法使用现有的数据库或者新建的数据库创建一个数据库对象。
+ transaction：这个方法让我们能够控制一个事务，以及基于这种情况执行提交或者回滚。
+ executeSql：这个方法用于执行实际的 SQL 查询。

openDatabase() 方法对应的五个参数说明：

+ 数据库名称
+ 版本号
+ 描述文本
+ 数据库大小
+ 创建回调

transaction执行数据库操作，操作内容就是正常的数据库的增删改查。

executeSql是执行具体的sql

```sql
var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);
db.transaction(function (tx) {
  tx.executeSql('CREATE TABLE IF NOT EXISTS LOGS (id unique, log)');
  tx.executeSql('INSERT INTO LOGS (id, log) VALUES (1, "菜鸟教程")');
  tx.executeSql('INSERT INTO LOGS (id,log) VALUES (?, ?)', [e_id, e_log]); //使用外部变量，执行时会将变量数组中的值依次替换前边的问号位置
    tx.executeSql('SELECT * FROM LOGS', [], function (tx, results) {
        var len = results.rows.length, i; msg = "<p>查询记录条数: " + len + "</p>"; document.querySelector('#status').innerHTML += msg; for (i = 0; i < len; i++){ msg = "<p><b>" + results.rows.item(i).log + "</b></p>"; document.querySelector('#status').innerHTML += msg; }
    }, null); //查询和回调
  tx.executeSql('DELETE FROM LOGS  WHERE id=1'); //删除
  tx.executeSql('DELETE FROM LOGS WHERE id=?', [id]);
  tx.executeSql('UPDATE LOGS SET log=\'www.w3cschool.cc\' WHERE id=2'); //更新
  tx.executeSql('UPDATE LOGS SET log=\'www.w3cschool.cc\' WHERE id=?', [id]);
});
```

基本操作与实际数据库操作基本一致。

最终的数据去向，我理解为只是做临时存储和大型网站的业务运行存储缓存的作用，页面刷新后该库就不存在了。而其本身与关系数据库的概念比较相似。

> IndexedDB

诞生背景：

　　随着浏览器的功能不断增强，越来越多的网站开始考虑，将大量数据储存在客户端，这样可以减少从服务器获取数据，直接从本地获取数据。现有的浏览器数据储存方案，都不适合储存大量数据：Cookie 的大小不超过4KB，且每次请求都会发送回服务器；LocalStorage 在 2.5MB 到 10MB 之间（各家浏览器不同），而且不提供搜索功能，不能建立自定义的索引。所以，需要一种新的解决方案，这就是 IndexedDB 诞生的背景。

 

　　IndexedDB是浏览器提供的本地数据库， 允许储存大量数据，提供查找接口，还能建立索引。这些都是 LocalStorage 所不具备的。就数据库类型而言，IndexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。

IndexedDB 具有以下特点：

（1）键值对储存。 IndexedDB 内部采用对象仓库（object store）存放数据。所有类型的数据都可以直接存入，包括 JavaScript 对象。对象仓库中，数据以"键值对"的形式保存，每一个数据记录都有对应的主键，主键是独一无二的，不能有重复，否则会抛出一个错误。

（2）异步。 IndexedDB 操作时不会锁死浏览器，用户依然可以进行其他操作，这与 LocalStorage 形成对比，后者的操作是同步的。异步设计是为了防止大量数据的读写，拖慢网页的表现。

（3）支持事务。 IndexedDB 支持事务（transaction），这意味着一系列操作步骤之中，只要有一步失败，整个事务就都取消，数据库回滚到事务发生之前的状态，不存在只改写一部分数据的情况。

（4）同源限制 IndexedDB 受到同源限制，每一个数据库对应创建它的域名。网页只能访问自身域名下的数据库，而不能访问跨域的数据库。

（5）储存空间大 IndexedDB 的储存空间比 LocalStorage 大得多，一般来说不少于 250MB，甚至没有上限。

（6）支持二进制储存。 IndexedDB 不仅可以储存字符串，还可以储存二进制数据（ArrayBuffer 对象和 Blob 对象）。

基本操作：

（1）打开数据库

使用 IndexedDB 的第一步是打开数据库，使用indexedDB.open()方法。

这个方法接受两个参数，第一个参数是字符串，表示数据库的名字。如果指定的数据库不存在，就会新建数据库。第二个参数是整数，表示数据库的版本。如果省略，打开已有数据库时，默认为当前版本；新建数据库时，默认为1。

indexedDB.open()方法返回一个 IDBRequest 对象。这个对象通过三种事件error、success、upgradeneeded，处理打开数据库的操作结果。

（2）新建数据库

```typescript
var db;
var objectStore;
var request = window.indexedDB.open(databaseName, version);

request.onerror = function (event) {}
request.onsuccess = function (event) {
    db = request.result//可以拿到数据库对象
}
//如果指定的版本号，大于数据库的实际版本号，就会发生数据库升级事件upgradeneeded
request.onupgradeneeded = function (event) {
    db = event.target.result;
    if (!db.objectStoreNames.contains('person')) {//判断是否存在
        objectStore = db.createObjectStore('person', { keyPath: 'id' });
//自动生成主键db.createObjectStore(
//  'person',
//  { autoIncrement: true }
//);
    }
    //新建索引，参数索引名称、索引所在的属性、配置对象
    objectStore.createIndex('email', 'email', { unique: true });
}
```

（3）新增数据

在以上操作的基础上，需要新建一个事务。新建时必须指定表格名称和操作模式（"只读"或"读写"）。新建事务以后，通过IDBTransaction.objectStore(name)方法，拿到 IDBObjectStore 对象，再通过表格对象的add()方法，向表格写入一条记录。

写入操作是一个异步操作，通过监听连接对象的success事件和error事件，了解是否写入成功。

```typescript
function add() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .add({ id: 1, name: '张三', age: 24, email: 'zhangsan@example.com' });

  request.onsuccess = function (event) {
    console.log('数据写入成功');
  };

  request.onerror = function (event) {
    console.log('数据写入失败');
  }
}

add();
```

（4）读取数据

objectStore.get()方法用于读取数据，参数是主键的值。

```typescript
function read() {
   var transaction = db.transaction(['person']);
   var objectStore = transaction.objectStore('person');
   var request = objectStore.get(1);

   request.onerror = function(event) {
     console.log('事务失败');
   };

   request.onsuccess = function( event) {
      if (request.result) {
        console.log('Name: ' + request.result.name);
        console.log('Age: ' + request.result.age);
        console.log('Email: ' + request.result.email);
      } else {
        console.log('未获得数据记录');
      }
   };
}

read();
```

（5）遍历数据

遍历数据表格的所有记录，要使用指针对象 IDBCursor。openCursor()方法是一个异步操作，所以要监听success事件。

```typescript
function readAll() {
  var objectStore = db.transaction('person').objectStore('person');

   objectStore.openCursor().onsuccess = function (event) {
     var cursor = event.target.result;

     if (cursor) {
       console.log('Id: ' + cursor.key);
       console.log('Name: ' + cursor.value.name);
       console.log('Age: ' + cursor.value.age);
       console.log('Email: ' + cursor.value.email);
       cursor.continue();
    } else {
      console.log('没有更多数据了！');
    }
  };
}

readAll();
```

（6）数据更新

IDBObject.put()方法。

```typescript
function update() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });

  request.onsuccess = function (event) {
    console.log('数据更新成功');
  };

  request.onerror = function (event) {
    console.log('数据更新失败');
  }
}

update();
```

（7）数据删除

IDBObjectStore.delete()方法用于删除记录。

```typescript
function remove() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .delete(1);

  request.onsuccess = function (event) {
    console.log('数据删除成功');
  };
}

remove();
```
（8）索引的使用

添加索引后可以使用索引查询数据

```typescript
var transaction = db.transaction(['person'], 'readonly');
var store = transaction.objectStore('person');
var index = store.index('name');
var request = index.get('李四');

request.onsuccess = function (e) {
  var result = e.target.result;
  if (result) {
    // ...
  } else {
    // ...
  }
}
```
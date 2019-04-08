比较纠结的问题就是再使用mysql-nodejs中配置连接池的时候，连接数迟迟不能释放，官网给的Demo文档如下：

```mysql
var mysql = require('mysql');
var pool  = mysql.createPool(...);

pool.getConnection(function(err, connection) {
  if (err) throw err; // not connected!

  // Use the connection
  connection.query('SELECT something FROM sometable', function (error, results, fields) {
    // When done with the connection, release it.
    connection.release();

    // Handle error after the release.
    if (error) throw error;

    // Don't use the connection here, it has been returned to the pool.
  });
});
```

但是发现，无论是`release`或者`destory`似乎都没有影响连接数。查找了各种文档也没有明确答案，最终还是看这个驱动的源码中
在文件包`mysql/lib/Pool.js`中


```mysql
Pool.prototype.releaseConnection = function releaseConnection(connection) {

  if (this._acquiringConnections.indexOf(connection) !== -1) {
    // connection is being acquired
    return;
  }

  if (connection._pool) {
    if (connection._pool !== this) {
      throw new Error('Connection released to wrong pool');
    }

    if (this._freeConnections.indexOf(connection) !== -1) {
      // connection already in free connection pool
      // this won't catch all double-release cases
      throw new Error('Connection already released');
    } else {
      // add connection to end of free queue
      this._freeConnections.push(connection);
      this.emit('release', connection);
    }
  }

  if (this._closed) {
    // empty the connection queue
    this._connectionQueue.splice(0).forEach(function (cb) {
      var err = new Error('Pool is closed.');
      err.code = 'POOL_CLOSED';
      process.nextTick(function () {
        cb(err);
      });
    });
  } else if (this._connectionQueue.length) {
    // get connection with next waiting callback
    this.getConnection(this._connectionQueue.shift());
  }
};
```

难道是`releaseConnection`，最后试验发现果然啊。代码回复正常，查询后闲置连接数准时回收释放。

```mysql
var mysql = require('mysql');
var pool  = mysql.createPool(...);

pool.getConnection(function(err, connection) {
  if (err) throw err; // not connected!

  // Use the connection
  connection.query('SELECT something FROM sometable', function (error, results, fields) {
    // When done with the connection, release it.
    connection.releaseConnection();

    // Handle error after the release.
    if (error) throw error;

    // Don't use the connection here, it has been returned to the pool.
  });
});
```

想一想文档都这么不准确，难道问题产生，请知晓！！
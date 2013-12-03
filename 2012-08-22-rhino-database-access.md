---
layout: article
title: JavaScript通过JDBC访问数据库
date: 2012-08-22 09:20
category: JS数据库
excerpt:
  Rhino是Mozilla开发的纯Java实现的JS引擎，已经包含在Java 6里。
  通过Rhino，我们可以在JS中直接调用Java库，这里演示通过JDBC访问数据库。
---

提起服务器端 JavaScript，很多人第一反应都是 Node.js。其实 Java 6 开始包含 Script Engine，其中就自带了一个“阉割版”的 Mozilla Rhino - 纯 Java 实现的 JavaScript 解释器。

使用 jrunscript 就能启动这个解释器。使用 Rhino 的好处是你能使用 JavaScript 语言做开发，但又能使用现成的浩瀚的 Java 库！而且支持编译成 class 文件。

我以连接 Sqlite 数据库为例子抛砖引玉。首先创建一个 sqlite 数据库：

```bash
sqliteλ sqlite3 user.db
SQLite version 3.7.3
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .header on
sqlite> select * from users;
id|name|age
1|Joe|24
2|redraiment|24
3|Kewell|30
sqlite> .quit
sqliteλ
```

JS 封装 JDBC

```javascript
var with_connection = function(url) {
    var connection = java.sql.DriverManager.getConnection(url);
    for (var argc = 1; argc < arguments.length; argc++) {
        arguments[argc].call(connection);
    }
    connection.close();
};

var with_query = function(sql, fn) {
    return function() {
        var call = this.prepareStatement(sql);
        var rs = call.executeQuery();
        var meta = rs.getMetaData();
        var list = [];
        while (rs.next()) {
            var o = {};
            for (var i = 1; i <= meta.getColumnCount(); i++) {
                o[meta.getColumnName(i)] = rs.getObject(i);
            }
            list.push(o);
        }
        rs.close();
        call.close();
        return fn(list);
    };
};
```

使用：读取表信息

```javascript
new org.sqlite.JDBC();
with_connection(
    'jdbc:sqlite:user.db',
    with_query('select * from users', function(rs) {
        for (var row = 0; row < rs.length; row++) {
            for (var columnName in rs[row]) {
                printf('%s => %s\n', columnName, rs[row][columnName]);
            }
            print('\n');
        }
    })
);
```

执行结果

```bash
sqliteλ jrunscript -cp "$CLASSPATH:$PWD/sqlite-jdbc-3.7.2.jar" jdbc.js 
id => 1
name => Joe
age => 24

id => 2
name => redraiment
age => 24

id => 3
name => Kewell
age => 30

sqliteλ
```
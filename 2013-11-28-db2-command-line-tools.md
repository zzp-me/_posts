---
layout: article
title: db2命令
category: DB2
date: 2013-11-28 15:56:20
excerpt: db2命令行环境配置
---

```bash
export PATH=$PATH:$DB2_HOME/bin
```

其中DB2_HOME是DB2 Client所在的目录

# 连接数据库

```bash
# 查看可用DB
db2 list db directory
# 连接DB，其中$DB_DIR由上条命令获得
db2 connect to $DB_DIR user $USER using $UNIX_PASSWORD
```

# 导出查询结果为csv文件

```bash
db2 export to data.csv of DEL select \* from table
```

# 从csv文件中导入数据到数据库

```bash
db2 import from data.csv of DEL insert into table
```

# 执行脚本

即批量执行代码，通常用于创建/更新SP、table、function、view等。

```bash
db2 -td^ -svf script.sql
```

其中 -td^ 指定分隔符用“^”而不是用默认的“;”，这在创建SP时使用。

# 断开连接

```bash
db2 quit
```

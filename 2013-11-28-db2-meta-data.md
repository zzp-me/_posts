---
layout: article
title: 查看元数据
category: DB2
date: 2013-11-28 16:15:33
excerpt: 查看版本信息，所有表、列、函数……的方法
---

# 查看版本信息

```sql
select * from sysibmadm.env_inst_info
```

# 查看所有表

```sql
select * from syscat.tables
```

# 查看所有列

```sql
select * from syscat.columns
```

# 查看所有函数

```sql
select * from syscat.functions
```

# 查看所有存储过程

```sql
select
  RoutineName,
  text
from
  syscat.routines
where
  RoutineName = 'XXX'
```

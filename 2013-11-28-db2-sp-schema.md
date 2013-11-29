---
layout: article
title: APS SP设置schema
category: DB2
date: 2013-11-28 15:43:44
---

DB2需要同时设置schema和current path，否则访问不到某些函数或SP。

```sql
set schema $SCHEMA_NAME;
set current path PATH_NAME1,PATH_NAME2;
```

---
layout: article
title: 获取相邻行的内容
category: DB2
date: 2013-11-28 16:55:26
excerpt: 查询时，获取当前行的上一行/下一行的数据
---

DB2提供了两个函数，允许在查询时获取上一行/下一行的数据

# 往上找

`LAG(表达式或字段, 偏移量, 默认值, IGNORE NULLS或RESPECT NULLS)`

# 往下找

`LEAD(表达式或字段, 偏移量, 默认值, IGNORE NULLS或RESPECT NULLS)`

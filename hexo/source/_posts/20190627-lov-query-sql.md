---
title: 通过会话SID取得LOV查询脚本
date: 2019-06-27 18:38:43
tags:
- Oracle
- E-Business Suite
- SQL
categories:
- Oracle
- E-Business Suite
---
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/20190627183706.png)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/20190627183733.png)

**注意** *（如果是多节点服务器，请确认前端与后台的匹对关系）* 输入取得的会话SID
```sql
  SELECT T.SQL_TEXT
    FROM V$SQLTEXT_WITH_NEWLINES T, V$SESSION S
   WHERE 1 = 1 AND S.PREV_SQL_ADDR = T.ADDRESS 
     AND S.SID = &P_SID --369
ORDER BY T.PIECE;
```



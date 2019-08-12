---
title: 开发FORM程序异常提示窗口
date: 2019-06-27 12:53:41
tags:
- Oracle
- E-Business Suite
- Form
categories:
- Oracle
- E-Business Suite
- Form
---
编写FORM程序的PL/SQL程序包产出异常情况时，弹出一个异常提示窗口。
```sql
DECLARE
BEGIN
app_exception.raise_exception (
  exception_type => 'APP'
, exception_code => -20001
, exception_text => 'OTHERS EXCETPITON ' || CHR (13) || TO_CHAR (DBMS_UTILITY.format_error_backtrace));
END;
```
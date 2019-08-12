---
title: PL/SQL统计运行的时间器
date: 2019-06-27 14:58:26
tags:
- Oracle
- PL/SQL
categories:
- Oracle
- PL/SQL
---
在遇到一些较为复杂的项目时，需要做技术设计，功能函数封装等，随之而来模块出现性能问题，故在编写PL/SQL程式时，尽量加上统计运行的时间器，方便性能分析提供依据。
```java
CREATE OR REPLACE PACKAGE timer AUTHID current_user IS
    PROCEDURE start_timer;

    PROCEDURE show_elapsed (
      program_name IN VARCHAR2
    );

END;
/

CREATE OR REPLACE PACKAGE BODY timer IS

    c_time_gap     NUMBER : = power(2, 32);
    l_start_time   PLS_INTEGER;

    PROCEDURE start_timer IS
    BEGIN
        l_start_time : = dbms_utility.get_time();
    END;

    PROCEDURE show_elapsed (
      program_name IN VARCHAR2
    ) AS
        l_end_time VARCHAR2(100);
    BEGIN
        l_end_time : = MOD(dbms_utility.get_time - l_start_time + c_time_gap, c_time_gap) / 100;
        dbms_output.put_line(program_name || ' has elapsed time '|| l_end_time|| ' s.');
    END;

END;
/
```


```sql
--example:
begin
pro_timing.start_timer;
dbms_output.put_line('this is a test');
dbms_lock.sleep(2);
pro_timing.show_elapsed('test program');
end;

--Result:
--this is a test
--test program has elapsed time 2 s.
```
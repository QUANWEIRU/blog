---
title: R12.x Accounting Calendar template
date: 2019-06-26 19:57:29
tags:
- Oracle
- E-Business Suite
- R12.x
- SQL
- Dataload
categories:
- Oracle
- E-Business Suite
---
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/20190626200402.png)
```SQL
--parameters:
--year : start year
--month: generate months
select 
--(case when mod(LEVEL,12) <>0 then mod(LEVEL,12) else  12 end)
--, (case when mod(LEVEL,12) <> 0 then floor(level/12) else floor(level/12)-1 end)
 initcap (to_char (to_date (lpad ((case when mod(level,12) <>0 then mod(level,12) else  12 end), 2, '0'), 'mm'), 'MON', 'NLS_DATE_LANGUAGE=AMERICAN')) prefix
, 'Month' type
, to_char(to_number(:year) +  (case when mod(level,12) <> 0 then floor(level/12) else floor(level/12)-1 end))  year
, to_char (to_date (lpad (to_char(case when mod(level,12) <>0 then mod(level,12) else  12 end), 2, '0'), 'mm'), 'q') quarter
, (case when mod(level,12) <>0 then mod(level,12) else  12 end) num
, upper (to_char (to_date (to_char(to_number(:year) +  (case when mod(level,12) <> 0 then floor(level/12) else floor(level/12)-1 end)) || '-' || lpad ((case when mod(level,12) <>0 then mod(level,12) else  12 end), 2, '0'), 'yyyy-mm'), 'dd-mon-yyyy')) date_from
, upper (to_char (last_day (to_date (to_char(to_number(:year) +  (case when mod(level,12) <> 0 then floor(level/12) else floor(level/12)-1 end)) || '-' || lpad ((case when mod(level,12) <>0 then mod(level,12) else  12 end), 2, '0'), 'yyyy-mm')), 'dd-mon-yyyy')) date_to
, initcap (to_char (to_date (to_char(to_number(:year) +  (case when mod(level,12) <> 0 then floor(level/12) else floor(level/12)-1 end)) || '-' || lpad ((case when mod(level,12) <>0 then mod(level,12) else  12 end), 2, '0'), 'yyyy-mm'), 'mon-yy')) name
from dual
connect by level <= :month
union all
select 'Adj' prefix
, 'Month' type
, to_char (to_number(:year) + level - 1) year
, '4' quarter
, 13 num
, upper (to_char (last_day (to_date (to_char (to_number(:year) + level - 1)  || '-12', 'yyyy-mm')), 'dd-mon-yyyy')) date_from
, upper (to_char (last_day (to_date (to_char (to_number(:year) + level - 1)  || '-12', 'yyyy-mm')), 'dd-mon-yyyy')) date_to
, 'Adj-' || to_char (to_date (to_char (to_number(:year) + level - 1) , 'yyyy'), 'yy') name
from dual
connect by level <= floor(:month/12)
order by year, num asc;
```
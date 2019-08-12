---
title: How to Achieve R12 Multi Org for Custom Table
date: 2019-06-28 11:12:02
tags:
- Oracle
- E-Business Suite
- PL/SQL
categories:
- Oracle
- E-Business Suite
---
We can achieve the Multi Org functionality for custom table with follow the below steps:

1. Create a custom table in custom schema, grant to user APPS, create synonyms for this table and insert some data.
```sql
CREATE TABLE cux.cux_inv_all (
    inv_num     VARCHAR2(30),
    inv_date    DATE,
    org_id      NUMBER);

GRANT ALL ON cux.cux_inv_all TO apps;
CREATE OR REPLACE SYNONYM apps.cux_inv_all FOR cux.cux_inv_all;
CREATE OR REPLACE SYNONYM apps.cux_inv FOR cux.cux_inv_all;
```

2. Apply RLS (Row Level Security) to CUX_INV synonym using standard Virtual Private Database (VPD) policy function MO_GLOBAL.ORG_SECURITY and DBMS_RLS.ADD_POLICY API.
```sql
BEGIN
dbms_rls.add_policy(
    object_schema     => 'APPS'
    , object_name       => 'CUX_INV'
    , policy_name       => 'ORG_SEG'
    , function_schema   => 'APPS'
    , policy_function   => 'MO_GLOBAL.ORG_SECURITY'
    , statement_types   => 'SELECT, INSERT, UPDATE, DELETE'
    , update_check      => true
    , enable            => true);
END;
```

3. Check the policy already added or not in ALL_POLICIES view. If added, it will return the data.
```sql
SELECT *
FROM   all_policies
WHERE  object_name='CUX_INV';
```

4. Get the data for which org_id initialized by using MO_GLOBAL.SET_POLICY_CONTEXT procedure.
* Without initialize the org_id
```sql
select * from cux_inv;
```
* Initialize org_id 204
```sql
exec mo_global.set_policy_context('S', 204);

select * from cux_inv;
```

5. Revoke the policy using DBMS_RLS.DROP_POLICY API. When you query to ALL_POLICIES view will get the result no row returned.
```sql
EXEC DBMS_RLS.DROP_POLICY('APPS','CUX_INV','ORG_SEC');
```
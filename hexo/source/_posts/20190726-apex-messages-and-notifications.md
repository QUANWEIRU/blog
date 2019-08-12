---
title: Mastering APEX Messages and Notifications
date: 2019-07-26 11:07:10
tags:
- Oracle
- Oracle APEX
categories:
- Oracle
- Oracle APEX
---

# AGENDA

+ Success Message
+ Prepared/Formatted message
+ Validation/Error message
+ Client side message
+ Common problems with message

## PL/SQL Success Message

+ Add success message to page processes
``` bash
Toggle message display in branch
```

+ Use subsititutions for dynamic messages
``` bash
&P6_ENAME. Updated!
```

+ Create a dynamic message in pl/sql
``` bash
apex_application.g_print_success_message := 'Salary doubled equals ' || (:P6_SAL * 2) || '!'
```

## What is a Prepared Message?

+ Predefined message with dynamic placeholders
```bash
Easy to read at a glance
Possible to create messages for multiple languages
```

+ Where can these be created?
```bash
Shared Components >> Text Messages
Defined in your own table or code
```

+ Messages using apex_string.format *(Supports both inplicit and explicit references)*
> **Implicit reference to parameters**
('Hello %s, how is the weather in %s?', 'Tyson', 'Huntsville')
=> Hello Tyson, how is the weather in Huntsville? 
> 
> **Explicit reference to parameters**
> ***Remember parameters references start with 0!***
('Hello %1, how is the weather in %0?', 'Huntsville', 'Tyson')
=> Hello Tyson, how is the weather in Huntsville? 
```bash
declare
    c_button_template   constant varchar2(32767) := '<button type="button" title="%0 %1" class="t-Button %1">%0</button>';
    l_button_classes apex_t_varchar2 := apex_string.split('u-hot,u-warning,u-success', ',');
begin
    sys.htp.p(apex_string.format(c_button_template, 'Template Button'));

    for i in 1 .. l_button_classes.count
    loop
        sys.htp.p(apex_string.format(c_button_template, 'Template Button', l_button_classes(i)));
    end loop;
end;
```
+ Messages using apex_lang.message *(Only supports explicit parameter references)*
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B1A29D2C5-7FAD-5706-76B7-53D1AE6BA7F5%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B61334F57-1362-C3C3-EA20-5E3DB5DCD8D9%7D.jpg)

+ apex_debug

## Validation and Error Message

+ apex_error    package is increditbly helpful
+ 1 error = 1 validation message
+ Define procedures to run many validations
+ Do not rely no the error handling function (we can create a better user experience)

![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7BDDBD8C67-BBDD-956C-0AD9-89A417A25D85%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B9C5BF49A-ABA3-5B2C-71E4-2F5172D090D4%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7BB946ABD2-6815-5908-CA44-131CBAB8F01D%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B94514E28-7A9C-059E-59C8-A7CDA9AC3A91%7D.jpg)
```bash
is_valid_emp(
    p_empno => :P51_EMPNO,
    p_ename => :P51_ENAME,
    p_sal   => :P51_SAL,
    p_job   => :P51_JOB,
    p_comm  => :P51_COMM,
    p_items => apex_util.string_to_table('P51_ENAME:P51_SAL:P51_JOB:P51_COMM')
);
return true;
```
```bash
create or replace procedure is_valid_emp(
    p_empno emp.empno%type,
    p_ename emp.ename%type,
    p_sal   emp.sal%type,
    p_job   emp.job%type,
    p_comm  emp.comm%type,
    p_items apex_application_global.vc_arr2
)
is
    c_ename_pos constant pls_integer := 1;
    c_sal_pos   constant pls_integer := 2;
    c_job_pos   constant pls_integer := 3;
    c_comm_pos  constant pls_integer := 4;

    l_page_id   pls_integer := nv('APP_PAGE_ID');
begin

    for error in (select count(*) from emp where ename = p_ename and empno != p_empno)
    loop
        apex_error.add_error(
            p_message           => 'Ename already exists!',
            p_display_location  => apex_error.c_inline_with_field_and_notif,
            p_page_item_name    => p_items(c_ename_pos)
        );
    end loop;

    if p_sal > 5000 then
        apex_error.add_error(
            p_message           => 'Salary must be less than or equal to 5000',
            p_display_location  => apex_error.c_inline_with_field_and_notif,
            p_page_item_name    => p_items(c_sal_pos)
        );
    end if;

    if p_comm > 500 then
        apex_error.add_error(
            p_message           => 'Comm must be less than or equal to 500',
            p_display_location  => apex_error.c_inline_with_field_and_notif,
            p_page_item_name    => p_items(c_sal_pos)
        );
    end if;

    if p_job != 'SALESMAN' and nvl(p_comm, 0) > 0 then
        apex_error.add_error(
            p_message           => 'Only Salesman can have a commission',
            p_display_location  => apex_error.c_inline_with_field_and_notif,
            p_page_item_name    => p_items(c_comm_pos)
        );
    end if;
end is_valid_emp;
```

## Error Handling Function

+ Wonderful general purpose error handler
+ Only runs when something went wrong
+ Can create frustrating user experience
+ Still an incredibly helpful fallback

```bash
create or replace function apex_error_handling_example (
    p_error in apex_error.t_error )
    return apex_error.t_error_result
is
    l_result          apex_error.t_error_result;
    l_reference_id    number;
    l_constraint_name varchar2(255);
begin
    l_result := apex_error.init_error_result (
                    p_error => p_error );
 
    -- If it's an internal error raised by APEX, like an invalid statement or
    -- code which cannot be executed, the error text might contain security sensitive
    -- information. To avoid this security problem rewrite the error to
    -- a generic error message and log the original error message for further
    -- investigation by the help desk.

   if p_error.is_internal_error then
        -- mask all errors that are not common runtime errors (Access Denied
        -- errors raised by application / page authorization and all errors
        -- regarding session and session state)
        if not p_error.is_common_runtime_error then
            -- log error for example with an autonomous transaction and return
            -- l_reference_id as reference#
            -- l_reference_id := log_error (
            --                       p_error => p_error );
            --
 
            -- Change the message to the generic error message which doesn't expose
            -- any sensitive information.
            l_result.message      := 'An unexpected internal application error has occurred. '||
                                        'Please get in contact with XXX and provide '||
                                        'reference# '||to_char(l_reference_id, '999G999G999G990')||
                                        ' for further investigation.';
            l_result.additional_info := null;
        end if;
    else
        -- Always show the error as inline error
        -- Note: If you have created manual tabular forms (using the package
        --       apex_item/htmldb_item in the SQL statement) you should still
        --       use "On error page" on that pages to avoid loosing entered data
        l_result.display_location := case
                                       when l_result.display_location = apex_error.c_on_error_page then apex_error.c_inline_in_notification
                                       else l_result.display_location
                                     end;
 
        -- If it's a constraint violation like
        --
        --   -) ORA-00001: unique constraint violated
        --   -) ORA-02091: transaction rolled back (-> can hide a deferred constraint)
        --   -) ORA-02290: check constraint violated
        --   -) ORA-02291: integrity constraint violated - parent key not found
        --   -) ORA-02292: integrity constraint violated - child record found
        --
        -- try to get a friendly error message from our constraint lookup configuration.
        -- If the constraint in our lookup table is not found, fallback to
        -- the original ORA error message.
        if p_error.ora_sqlcode in (-1, -2091, -2290, -2291, -2292) then
            l_constraint_name := apex_error.extract_constraint_name (
                                     p_error => p_error );
        
            begin
                select message
                  into l_result.message
                  from constraint_lookup
                 where constraint_name = l_constraint_name;
            exception when no_data_found then null; -- not every constraint has to be in our lookup table
            end;
        end if;
        
        -- If an ORA error has been raised, for example a raise_application_error(-20xxx, '...')
        -- in a table trigger or in a PL/SQL package called by a process and the 
        -- error has not been found in the lookup table, then display
        -- the actual error text and not the full error stack with all the ORA error numbers.
        if p_error.ora_sqlcode is not null and l_result.message = p_error.message then
            l_result.message := apex_error.get_first_ora_error_text (
                                    p_error => p_error );
        end if;
 
        -- If no associated page item/tabular form column has been set, use
        -- apex_error.auto_set_associated_item to automatically guess the affected
        -- error field by examine the ORA error for constraint names or column names.
        if l_result.page_item_name is null and l_result.column_alias is null then
            apex_error.auto_set_associated_item (
                p_error        => p_error,
                p_error_result => l_result );
        end if;
    end if;
 
    return l_result;
end apex_error_handling_example;
```

## Client side message in Javascript

+ ### apex.message namespace is awesome!
+ success messages
+ error messages
+ alerts

+ ### Use *setThemeHooks* to extend functionality
+ beforeShow lifecycle event
+ afterShow lifecycle event

![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B12C43EAA-6AE1-B601-6972-8A3DF90F874D%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7BF38E4EA7-A04A-A5BA-348D-7D0AC881093A%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B887B4A11-FAB1-5036-6DFB-49EE4726DB22%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B4BB65958-BB3D-C571-6072-401B889B3D84%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B488B448C-BBAA-1D88-2A59-9129CB54873D%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B4CCB674B-C368-68A2-264A-7DD43A4B84D0%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B3915E29C-C1B0-CA0D-0121-5F7B5574B456%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7BA12B1A6F-C997-ADDF-184B-42E16D420F52%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B1474FA97-06CE-750E-7390-6369FEE48DF7%7D.jpg)
![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B8B1D5261-871C-4565-14BD-0CD9AD49BFB5%7D.jpg)


## Using Message Theme Hooks

![](https://raw.githubusercontent.com/QUANWEIRU/blog-images/master/%7B93BA5141-BD47-65ED-9D8C-984F6F16279E%7D.jpg)
```bash
(function(message){
    let duration = 5000;
    let messageTimeout = setTimeout(messsage.hidePageSuccess, duration);
    message.setThemeHooks({
        beforShow: function(pMsgType, pElement$){
            if(pMsgType === apex.message.TYPE.SUCCESS){
                clearTimeout(messageTimeout);
                messageTimeout = setTimeout(messsage.hidePageSuccess, duration);
            }
        }
    });
})(apex.message);
```

## Common Problems

+ Used in compiled/custom code
+ Limited options for presenting values
+ Fomratting parameters would be great
+ Named parameters would be awesom
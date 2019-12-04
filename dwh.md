
### Preface

This document contains our proposal for development standards for **oracle plsql**,
**oracle data integrator** and **general dwh concepts**. These standards, and the need for them are explained detailly
in related sections.

### The Need for Standards

In organizations, with the growth of the number of team memebers, a need for mutual understanding and communication
arises. Therefore standards plays important role in maintaining a healty organization. Standards make people
understand each other better and follow a simple pattern more quickly and easily.

We will cover some standards we propose for development and architecture in a data warehouse environment.

### Oracle PLSQL

#### Naming conventions:
  - **Case :** PLSQL is case insensitive, so like other case insensitive languages **`snake_case`** (`lowercase_words_separated_by_underscores`) is better.
  - **Package names** should be  `pkg_`
  - **Procedure names** should begin with `prc_`
  - **Function names** should begin with `fun_`
  - **Method variables :** Input method varaibles shoult begin with `i_`. Output method variables
    should begin with `o_`. Example:

    ```plsql
    procedure prc_my_load_task(i_start_data date, i_end_date date, o_result out number );
    ```
  - **Global variables** should begin with `gv_`
  - **Local variables** should begin with `v_`

#### PLSQL package layout:

  - **Use two spaces** indentation. **Never use tabs** for indentation.
    Depending on who and when you ask, a tab is worth either eight spaces, four spaces, two spaces, etc..
    So use spaces instead of tabs, the idiomatic SQL should be tab free. Code is written once but read many times,
    that's why readibility is very important.

  - Line comments should be used instead of block comments whenever possible.
    - ok
      ```sql
      -- some comment
      -- some other comment
      ```
    - not ok
      ```sql
      /*
        some comment
        some other coment
       */
      ```

  - Package documentation should be given at the top
    ```sql
    -----------------------------------
    -- This package is created for ...
    -- This package does following ...
    -----------------------------------
    ```

  - **Comments:** Good code should speak for itself, and most of the time you should let it do its own talking.
    If it’s obvious how someone would use your method—if the program needs no explanation—then don’t explain it. Above all, avoid boilerplate comments: Never put in a comment simply because you always put a comment there.

  - Global variables must be declared at the top of the package.

    ```plsql
    CREATE OR REPLACE PACKAGE BODY SCHEMA.PKG_PACKAGE_NAME
    is
      -----------------------------------
      -- Initialize Log Variables      --
      -----------------------------------
      gv_pkg        constant varchar2(100)  := 'SCHEMA.PKG_PACKAGE_NAME';  -- PLSQL Package Name
      gv_proc       varchar2(100);                                         -- Procedure Name
      gv_sql        long := '';

      -- example schemas that will be used in methods
      gv_sg_owner   varchar2(30) := 'SG';
      gv_stg_owner  varchar2(30) := 'TESTDWH';
      gv_dwh_owner  varchar2(30) := 'TESTDWH';
      gv_kpi_owner  varchar2(30) := 'KPI';
    ....
    ```

  - Procedures should include logging for the dml/ddl operations.

  - Procedures should handle and log exceptions as error messages
    eg.
    ```plsql
    begin
      -- method body
    exception when others then
      pl.logeer.error(sqlerrm);
      raise
    end
    ```
  - Procedures also should log success messages and statement that is run.
    eg.
    ```plsql
    begin
      -- method body
      gv_sql := '
        statement ...
      `;
      execute immediate gv_sql;
      pl.logger.success(gv_sql);
      commit; -- if dml
    exception when others then
      pl.logeer.error(sqlerrm);
      raise
    end
    ```
  - There should be **only one statement per procedure**.

  - An example package body can be;
  ```plsql
  CREATE OR REPLACE PACKAGE BODY SCHEMA_NAME.PKG_PACKAGE_NAME
  is
    -----------------------------------
    -- Initialize Log Variables      --
    -----------------------------------
    gv_pkg        constant varchar2(100)  := 'TESTDWH.PKG_PACKAGE_NAME';  -- PLSQL Package Name
    gv_proc       varchar2(100);                                          -- Procedure Name

    gv_sql        long := '';
    gv_high_date  constant date := to_date('2999.01.01','yyyy.mm.dd');

    -- Example schema names that is used in methods
    gv_sg_owner   varchar2(30) := 'SG';
    gv_stg_owner  varchar2(30) := 'TESTDWH';
    gv_dwh_owner  varchar2(30) := 'TESTDWH';
    gv_kpi_owner  varchar2(30) := 'KPI';

    PROCEDURE PRC_PROC_NAME(i_start_date date := trunc(sysdate-1), i_end_date date := trunc(sysdate)) IS
      ----------------------------------------------------------------------------
      v_trg_table varchar2(30) := 'TARGET_TABLE';
      ----------------------------------------------------------------------------
      v_src_table_01 varchar2(30) := 'SOURCE_TABLE_1';
      v_src_table_02 varchar2(30) := 'SOURCE_TABLE_2';
      ----------------------------------------------------------------------------
    BEGIN
      gv_proc := 'PRC_PROC_NAME';
      pl.logger := util.logtype.init(gv_pkg||'.'||gv_proc);
      pl.drop_table(gv_stg_owner, v_trg_table);

      gv_sql := '
        CREATE TABLE '||gv_stg_owner||'.'||v_trg_table||'
        parallel nologging compress
        AS
        SELECT
          ...
        FROM
          '||gv_stg_owner||'.'||v_src_table_01||' t1,
          '||gv_dwh_owner||'.'||v_src_table_02||' t2
        WHERE
          ....
        GROUP BY
          ...
      ';

      pl.enable_parallel_dml;
      execute immediate gv_sql;
      pl.logger.success(SQL%ROWCOUNT,gv_sql);

    EXCEPTION
      WHEN OTHERS THEN
        pl.logger.error(SQLCODE||' : '||SQLERRM, gv_sql);
        rollback;
        raise;
    END;

  END;
  ```


### Oracle Metadata

  - Target table names should be short clean and understandable
  - Use `T[#]_TABLE_NAME` or variants for temporary tables to dissociate them from target tables.
    eg. `T0010_SOME_TABLE`, `T0010SOME_TABLE`, `T$SOME_TABLE` etc. `T` in here shows it is a temp. table.
  - View names chould begin with `V` to show it is not a table but view.
  - Do not use views inside views/materialized-views unless you have to.
  - Do not use materialized-views inside views/materialized-views unless you have to.
  - Prefer packages and procedures over materialized views.
  - Use `under_score` naming convention for tables, columns, views, etc..

### Oracle Data Integrator

  - It is better to use mappings for one statement only but it is not as strict as plsql packages. The reason
    for not being strict is that, odi-mappings are restartable from the failed steps but plsql procs. are not.
  - Do not prefer pre-mapping and post-mapping sections in mappings because they are easy to skip, not seen
    by accident.
  - Each action whether it is a package, map, proc, proc calling plsql should be one scenario.
  - Use load plans to schedule scenarios.
  - Keep repsitory clean. Remove unused items and logs periodically.
  - Do not share passwords of the users that are being used by agents. If you need to share, instead,
      - Duplicate the user you gonna share
      - Change the duplicated users password
      - And share it.
      - when you are all done, remove the new user later on.
    This is because, when you share the password temporarily and want to replace the password later,
    you have to re-configure agents for the new password and restart them. Instead use the above suggested method.

### DWH in General

I want to share *zen of python* here; Because i think it is universal for most of the things.

- Beautiful is better than ugly.
- Explicit is better than implicit.
- Simple is better than complex.
- Complex is better than complicated.
- Flat is better than nested.
- Sparse is better than dense.
- Readability counts.
- Special cases aren't special enough to break the rules.
- Although practicality beats purity.
- Errors should never pass silently.
- Unless explicitly silenced.
- In the face of ambiguity, refuse the temptation to guess.
- There should be one-- and preferably only one --obvious way to do it.
- Although that way may not be obvious at first unless you're Dutch.
- Now is better than never.
- Although never is often better than *right* now.
- If the implementation is hard to explain, it's a bad idea.
- If the implementation is easy to explain, it may be a good idea.
- Namespaces are one honking great idea -- let's do more of those!

I also suggest you to take a look at [here](http://www.wikizero.biz/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvUHJpbmNpcGxlX29mX2xlYXN0X2FzdG9uaXNobWVudA) for the
*Principle of least astonishment*

For dwh specific rules;

- Do not use cursors for dml operations.
- Do not overuse subqueries
- Always check execution plans
- It is better to read one dataset **once** , make calculations and write the new dataset.
  For example;
  you have datasets `A, B, C`

  ```
  Do Not:
    ( read A,  read B(+) :join) => (calculate (a,b) ) => write A'
    ( read A', read C(+) :join) => (calculate (a',c)) => write A''
  Do Instead:
    ( read A, read B(+), read C(+) :join) =>(calculate (a,b,c)) => write A''
  ```
- Always make procedures restartable.
- Remember to clean temp objects for non-debugging runs.

- Do not use select statements inside column expressions unless you are using index;
```sql
  ------------------
  --! This is wrong
  ------------------
  select
    ...
    (select column from table_1 t1 where t1.id = t2.id) column_alias,
    ...
  from
    table_2 t2
  where
    ....
```






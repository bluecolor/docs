# PL/SQL Commons

  Contains common utility and logging methods.
  A simple proc that uses `pl` looks like the following;

```sql
  -- PL in action !
  --
  PROCEDURE PRC_PROC_NAME(<i_input_var vartype>, <o_output_var vartype> ) IS
  BEGIN
    gv_proc := 'PRC_PROC_NAME'; -- name of the procedure
    -- gv_pkg -- constant name of the package set globally once

    -- initialize logger here
    pl.logger := util.logtype.init(gv_pkg ||'.'||gv_proc);

    --------------------
    -- proc body here --
    --------------------
    -- gv_sql : global variable
    --
    gv_sql := '
      -- sql statement to execute
    ';
    execute immediate gv_sql;

    -- success message
    pl.logger.success(SQL%ROWCOUNT,gv_sql);

    -- !!! commit should be after success message
    commit;

  EXCEPTION
    WHEN OTHERS THEN
      -- error message
      pl.logger.error(SQLCODE || ' : ' ||SQLERRM);
      raise;
  END;
```

You can see the recent logs in `logs` table
```sql
select * from util.logs order by 3 desc;
```

### Installation

See the [repository](https://github.com/bluecolor/pl) for installation
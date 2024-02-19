# 9.25. 集合回傳函數

本節描述可能傳回多筆資料集合的函數。 此類別中使用最廣泛的函數是數列生成函數，如 [Table 9.65](set-returning-functions.md#table-9.65.-series-generating-functions) 和 [Table 9.66](set-returning-functions.md#table-9.66.-subscript-generating-functions) 所示。 其他更詳細的集合回傳函數在本手冊的其他章節也有說明。有關組合多個集合回傳函數的方法，請參閱[第 7.2.1.4 節](../queries/table-expressions.md#7.2.1.4.-zi-liao-biao-han-shu)。

#### **Table 9.65. Series Generating Functions**

| <p>Function</p><p>Description</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><code>generate_series</code> ( <em><code>start</code></em> <code>integer</code>, <em><code>stop</code></em> <code>integer</code> [, <em><code>step</code></em> <code>integer</code> ] ) → <code>setof integer</code></p><p><code>generate_series</code> ( <em><code>start</code></em> <code>bigint</code>, <em><code>stop</code></em> <code>bigint</code> [, <em><code>step</code></em> <code>bigint</code> ] ) → <code>setof bigint</code></p><p><code>generate_series</code> ( <em><code>start</code></em> <code>numeric</code>, <em><code>stop</code></em> <code>numeric</code> [, <em><code>step</code></em> <code>numeric</code> ] ) → <code>setof numeric</code></p><p>產生從 start 到 stop 的一系列數值，間隔為 step，其預設為 1。</p>                                                                          |
| <p><code>generate_series</code> ( <em><code>start</code></em> <code>timestamp</code>, <em><code>stop</code></em> <code>timestamp</code>, <em><code>step</code></em> <code>interval</code> ) → <code>setof timestamp</code></p><p><code>generate_series</code> ( <em><code>start</code></em> <code>timestamp with time zone</code>, <em><code>stop</code></em> <code>timestamp with time zone</code>, <em><code>step</code></em> <code>interval</code> [, <em><code>timezone</code></em> <code>text</code> ] ) → <code>setof timestamp with time zone</code></p><p>產生從 start 到 stop 的一系列數值，間隔為 step。在 with time zone 的時候，一天中的時間和夏令時調整是根據 timezone 參數指定的時區或目前 <a href="../../server-administration/server-configuration/client-connection-defaults.md#timezone-string">TimeZone</a> 設定（如果省略的話）所計算的。</p> |

When _`step`_ is positive, zero rows are returned if _`start`_ is greater than _`stop`_. Conversely, when _`step`_ is negative, zero rows are returned if _`start`_ is less than _`stop`_. Zero rows are also returned if any input is `NULL`. It is an error for _`step`_ to be zero. Some examples follow:

```
SELECT * FROM generate_series(2,4);
 generate_series
-----------------
               2
               3
               4
(3 rows)

SELECT * FROM generate_series(5,1,-2);
 generate_series
-----------------
               5
               3
               1
(3 rows)

SELECT * FROM generate_series(4,3);
 generate_series
-----------------
(0 rows)

SELECT generate_series(1.1, 4, 1.3);
 generate_series
-----------------
             1.1
             2.4
             3.7
(3 rows)

-- this example relies on the date-plus-integer operator:
SELECT current_date + s.a AS dates FROM generate_series(0,14,7) AS s(a);
   dates
------------
 2004-02-05
 2004-02-12
 2004-02-19
(3 rows)

SELECT * FROM generate_series('2008-03-01 00:00'::timestamp,
                              '2008-03-04 12:00', '10 hours');
   generate_series
---------------------
 2008-03-01 00:00:00
 2008-03-01 10:00:00
 2008-03-01 20:00:00
 2008-03-02 06:00:00
 2008-03-02 16:00:00
 2008-03-03 02:00:00
 2008-03-03 12:00:00
 2008-03-03 22:00:00
 2008-03-04 08:00:00
(9 rows)

-- this example assumes that TimeZone is set to UTC; note the DST transition:
SELECT * FROM generate_series('2001-10-22 00:00 -04:00'::timestamptz,
                              '2001-11-01 00:00 -05:00'::timestamptz,
                              '1 day'::interval, 'America/New_York');
    generate_series
------------------------
 2001-10-22 04:00:00+00
 2001-10-23 04:00:00+00
 2001-10-24 04:00:00+00
 2001-10-25 04:00:00+00
 2001-10-26 04:00:00+00
 2001-10-27 04:00:00+00
 2001-10-28 04:00:00+00
 2001-10-29 05:00:00+00
 2001-10-30 05:00:00+00
 2001-10-31 05:00:00+00
 2001-11-01 05:00:00+00
(11 rows)
```

#### **Table 9.66. Subscript Generating Functions**

| <p>Function</p><p>Description</p>                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <p><code>generate_subscripts</code> ( <em><code>array</code></em> <code>anyarray</code>, <em><code>dim</code></em> <code>integer</code> ) → <code>setof integer</code></p><p>Generates a series comprising the valid subscripts of the <em><code>dim</code></em>'th dimension of the given array.</p>                                                                                                                                      |
| <p><code>generate_subscripts</code> ( <em><code>array</code></em> <code>anyarray</code>, <em><code>dim</code></em> <code>integer</code>, <em><code>reverse</code></em> <code>boolean</code> ) → <code>setof integer</code></p><p>Generates a series comprising the valid subscripts of the <em><code>dim</code></em>'th dimension of the given array. When <em><code>reverse</code></em> is true, returns the series in reverse order.</p> |

`generate_subscripts` is a convenience function that generates the set of valid subscripts for the specified dimension of the given array. Zero rows are returned for arrays that do not have the requested dimension, or if any input is `NULL`. Some examples follow:

```
-- basic usage:
SELECT generate_subscripts('{NULL,1,NULL,2}'::int[], 1) AS s;
 s
---
 1
 2
 3
 4
(4 rows)

-- presenting an array, the subscript and the subscripted
-- value requires a subquery:
SELECT * FROM arrays;
         a
--------------------
 {-1,-2}
 {100,200,300}
(2 rows)

SELECT a AS array, s AS subscript, a[s] AS value
FROM (SELECT generate_subscripts(a, 1) AS s, a FROM arrays) foo;
     array     | subscript | value
---------------+-----------+-------
 {-1,-2}       |         1 |    -1
 {-1,-2}       |         2 |    -2
 {100,200,300} |         1 |   100
 {100,200,300} |         2 |   200
 {100,200,300} |         3 |   300
(5 rows)

-- unnest a 2D array:
CREATE OR REPLACE FUNCTION unnest2(anyarray)
RETURNS SETOF anyelement AS $$
select $1[i][j]
   from generate_subscripts($1,1) g1(i),
        generate_subscripts($1,2) g2(j);
$$ LANGUAGE sql IMMUTABLE;
CREATE FUNCTION
SELECT * FROM unnest2(ARRAY[[1,2],[3,4]]);
 unnest2
---------
       1
       2
       3
       4
(4 rows)
```

When a function in the `FROM` clause is suffixed by `WITH ORDINALITY`, a `bigint` column is appended to the function's output column(s), which starts from 1 and increments by 1 for each row of the function's output. This is most useful in the case of set returning functions such as `unnest()`.

```
-- set returning function WITH ORDINALITY:
SELECT * FROM pg_ls_dir('.') WITH ORDINALITY AS t(ls,n);
       ls        | n
-----------------+----
 pg_serial       |  1
 pg_twophase     |  2
 postmaster.opts |  3
 pg_notify       |  4
 postgresql.conf |  5
 pg_tblspc       |  6
 logfile         |  7
 base            |  8
 postmaster.pid  |  9
 pg_ident.conf   | 10
 global          | 11
 pg_xact         | 12
 pg_snapshots    | 13
 pg_multixact    | 14
 PG_VERSION      | 15
 pg_wal          | 16
 pg_hba.conf     | 17
 pg_stat_tmp     | 18
 pg_subtrans     | 19
(19 rows)
```

\

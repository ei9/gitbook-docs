# dblink\_close

dblink\_close — closes a cursor in a remote database

### Synopsis

```
dblink_close(text cursorname [, bool fail_on_error]) returns text
dblink_close(text connname, text cursorname [, bool fail_on_error]) returns text
```

### Description

`dblink_close` closes a cursor previously opened with `dblink_open`.

### Arguments

_`connname`_

Name of the connection to use; omit this parameter to use the unnamed connection.

_`cursorname`_

The name of the cursor to close.

_`fail_on_error`_

If true (the default when omitted) then an error thrown on the remote side of the connection causes an error to also be thrown locally. If false, the remote error is locally reported as a NOTICE, and the function's return value is set to `ERROR`.

### Return Value

Returns status, either `OK` or `ERROR`.

### Notes

If `dblink_open` started an explicit transaction block, and this is the last remaining open cursor in this connection, `dblink_close` will issue the matching `COMMIT`.

### Examples

```
SELECT dblink_connect('dbname=postgres options=-csearch_path=');
 dblink_connect
----------------
 OK
(1 row)

SELECT dblink_open('foo', 'select proname, prosrc from pg_proc');
 dblink_open
-------------
 OK
(1 row)

SELECT dblink_close('foo');
 dblink_close
--------------
 OK
(1 row)
```

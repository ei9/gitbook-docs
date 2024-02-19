# CREATE SERVER

CREATE SERVER — define a new foreign server

## Synopsis

```
CREATE SERVER [IF NOT EXISTS] 
server_name
 [ TYPE '
server_type
' ] [ VERSION '
server_version
' ]
    FOREIGN DATA WRAPPER 
fdw_name

    [ OPTIONS ( 
option
 '
value
' [, ... ] ) ]
```

## Description

`CREATE SERVER`defines a new foreign server. The user who defines the server becomes its owner.

A foreign server typically encapsulates connection information that a foreign-data wrapper uses to access an external data resource. Additional user-specific connection information may be specified by means of user mappings.

The server name must be unique within the database.

Creating a server requires`USAGE`privilege on the foreign-data wrapper being used.

## Parameters

`IF NOT EXISTS`

Do not throw an error if a server with the same name already exists. A notice is issued in this case. Note that there is no guarantee that the existing server is anything like the one that would have been created.

`server_name`

The name of the foreign server to be created.

`server_type`

Optional server type, potentially useful to foreign-data wrappers.

`server_version`

Optional server version, potentially useful to foreign-data wrappers.

`fdw_name`

The name of the foreign-data wrapper that manages the server.

`OPTIONS (`

`option`

'

`value`

' \[, ... ] )

This clause specifies the options for the server. The options typically define the connection details of the server, but the actual names and values are dependent on the server's foreign-data wrapper.

## Notes

When using the[dblink](https://www.postgresql.org/docs/10/static/dblink.html)module, a foreign server's name can be used as an argument of the[dblink\_connect](https://www.postgresql.org/docs/10/static/contrib-dblink-connect.html)function to indicate the connection parameters. It is necessary to have the`USAGE`privilege on the foreign server to be able to use it in this way.

## Examples

Create a server`myserver`that uses the foreign-data wrapper`postgres_fdw`:

```
CREATE SERVER myserver FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host 'foo', dbname 'foodb', port '5432');
```

See[postgres\_fdw](https://www.postgresql.org/docs/10/static/postgres-fdw.html)for more details.

## Compatibility

`CREATE SERVER`conforms to ISO/IEC 9075-9 (SQL/MED).

## See Also

[ALTER SERVER](https://www.postgresql.org/docs/10/static/sql-alterserver.html)

,

[DROP SERVER](https://www.postgresql.org/docs/10/static/sql-dropserver.html)

,

[CREATE FOREIGN DATA WRAPPER](https://www.postgresql.org/docs/10/static/sql-createforeigndatawrapper.html)

,

[CREATE FOREIGN TABLE](https://www.postgresql.org/docs/10/static/sql-createforeigntable.html)

,

[CREATE USER MAPPING](https://www.postgresql.org/docs/10/static/sql-createusermapping.html)

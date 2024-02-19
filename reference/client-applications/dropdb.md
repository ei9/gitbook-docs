# dropdb

dropdb — remove aPostgreSQLdatabase

## Synopsis

`dropdb`\[`connection-option`...] \[`option`...]`dbname`

## Description

dropdbdestroys an existingPostgreSQLdatabase. The user who executes this command must be a database superuser or the owner of the database.

dropdbis a wrapper around theSQLcommand[DROP DATABASE](https://www.postgresql.org/docs/10/static/sql-dropdatabase.html). There is no effective difference between dropping databases via this utility and via other methods for accessing the server.

## Options

dropdbaccepts the following command-line arguments:

`dbname`

Specifies the name of the database to be removed.

`-e`

`--echo`

Echo the commands thatdropdbgenerates and sends to the server.

`-i`

`--interactive`

Issues a verification prompt before doing anything destructive.

`-V`

`--version`

Print thedropdbversion and exit.

`--if-exists`

Do not throw an error if the database does not exist. A notice is issued in this case.

`-?`

`--help`

Show help aboutdropdbcommand line arguments, and exit.

dropdbalso accepts the following command-line arguments for connection parameters:

`-h`

`host`

`--host=`

`host`

Specifies the host name of the machine on which the server is running. If the value begins with a slash, it is used as the directory for the Unix domain socket.

`-p`

`port`

`--port=`

`port`

Specifies the TCP port or local Unix domain socket file extension on which the server is listening for connections.

`-U`

`username`

`--username=`

`username`

User name to connect as.

`-w`

`--no-password`

Never issue a password prompt. If the server requires password authentication and a password is not available by other means such as a`.pgpass`file, the connection attempt will fail. This option can be useful in batch jobs and scripts where no user is present to enter a password.

`-W`

`--password`

Forcedropdbto prompt for a password before connecting to a database.

This option is never essential, sincedropdbwill automatically prompt for a password if the server demands password authentication. However,dropdbwill waste a connection attempt finding out that the server wants a password. In some cases it is worth typing`-W`to avoid the extra connection attempt.

`--maintenance-db=`

`dbname`

Specifies the name of the database to connect to in order to drop the target database. If not specified, the`postgres`database will be used; if that does not exist (or is the database being dropped),`template1`will be used.

## Environment

`PGHOST`

`PGPORT`

`PGUSER`

Default connection parameters

This utility, like most otherPostgreSQLutilities, also uses the environment variables supported bylibpq(see[Section 33.14](https://www.postgresql.org/docs/10/static/libpq-envars.html)).

## Diagnostics

In case of difficulty, see[DROP DATABASE](https://www.postgresql.org/docs/10/static/sql-dropdatabase.html)and[psql](https://www.postgresql.org/docs/10/static/app-psql.html)for discussions of potential problems and error messages. The database server must be running at the targeted host. Also, any default connection settings and environment variables used by thelibpqfront-end library will apply.

## Examples

To destroy the database`demo`on the default database server:

```
$ 
dropdb demo
```

To destroy the database`demo`using the server on host`eden`, port 5000, with verification and a peek at the underlying command:

```
$ 
dropdb -p 5000 -h eden -i -e demo
Database "demo" will be permanently deleted.
Are you sure? (y/n) 
y
DROP DATABASE demo;
```

## See Also

[createdb](https://www.postgresql.org/docs/10/static/app-createdb.html)

,

[DROP DATABASE](https://www.postgresql.org/docs/10/static/sql-dropdatabase.html)

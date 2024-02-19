# I. SQL 指令

本部分包含 PostgreSQL 支援的 SQL 指令的參考訊息。一般而言，「SQL」是指語言；內容包含了有關各標準的一致性和相容性。

連結將會連結至 PostgreSQL 官方使用手冊，本手冊連結請使用左側目錄。

**Table of Contents**

[ABORT](https://www.postgresql.org/docs/15/sql-abort.html) — abort the current transaction

[ALTER AGGREGATE](https://www.postgresql.org/docs/15/sql-alteraggregate.html) — change the definition of an aggregate function

[ALTER COLLATION](https://www.postgresql.org/docs/15/sql-altercollation.html) — change the definition of a collation

[ALTER CONVERSION](https://www.postgresql.org/docs/15/sql-alterconversion.html) — change the definition of a conversion

[ALTER DATABASE](https://www.postgresql.org/docs/15/sql-alterdatabase.html) — change a database

[ALTER DEFAULT PRIVILEGES](https://www.postgresql.org/docs/15/sql-alterdefaultprivileges.html) — define default access privileges

[ALTER DOMAIN](https://www.postgresql.org/docs/15/sql-alterdomain.html) — change the definition of a domain

[ALTER EVENT TRIGGER](https://www.postgresql.org/docs/15/sql-altereventtrigger.html) — change the definition of an event trigger

[ALTER EXTENSION](https://www.postgresql.org/docs/15/sql-alterextension.html) — change the definition of an extension

[ALTER FOREIGN DATA WRAPPER](https://www.postgresql.org/docs/15/sql-alterforeigndatawrapper.html) — change the definition of a foreign-data wrapper

[ALTER FOREIGN TABLE](https://www.postgresql.org/docs/15/sql-alterforeigntable.html) — change the definition of a foreign table

[ALTER FUNCTION](https://www.postgresql.org/docs/15/sql-alterfunction.html) — change the definition of a function

[ALTER GROUP](https://www.postgresql.org/docs/15/sql-altergroup.html) — change role name or membership

[ALTER INDEX](https://www.postgresql.org/docs/15/sql-alterindex.html) — change the definition of an index

[ALTER LANGUAGE](https://www.postgresql.org/docs/15/sql-alterlanguage.html) — change the definition of a procedural language

[ALTER LARGE OBJECT](https://www.postgresql.org/docs/15/sql-alterlargeobject.html) — change the definition of a large object

[ALTER MATERIALIZED VIEW](https://www.postgresql.org/docs/15/sql-altermaterializedview.html) — change the definition of a materialized view

[ALTER OPERATOR](https://www.postgresql.org/docs/15/sql-alteroperator.html) — change the definition of an operator

[ALTER OPERATOR CLASS](https://www.postgresql.org/docs/15/sql-alteropclass.html) — change the definition of an operator class

[ALTER OPERATOR FAMILY](https://www.postgresql.org/docs/15/sql-alteropfamily.html) — change the definition of an operator family

[ALTER POLICY](https://www.postgresql.org/docs/15/sql-alterpolicy.html) — change the definition of a row-level security policy

[ALTER PROCEDURE](https://www.postgresql.org/docs/15/sql-alterprocedure.html) — change the definition of a procedure

[ALTER PUBLICATION](https://www.postgresql.org/docs/15/sql-alterpublication.html) — change the definition of a publication

[ALTER ROLE](https://www.postgresql.org/docs/15/sql-alterrole.html) — change a database role

[ALTER ROUTINE](https://www.postgresql.org/docs/15/sql-alterroutine.html) — change the definition of a routine

[ALTER RULE](https://www.postgresql.org/docs/15/sql-alterrule.html) — change the definition of a rule

[ALTER SCHEMA](https://www.postgresql.org/docs/15/sql-alterschema.html) — change the definition of a schema

[ALTER SEQUENCE](https://www.postgresql.org/docs/15/sql-altersequence.html) — change the definition of a sequence generator

[ALTER SERVER](https://www.postgresql.org/docs/15/sql-alterserver.html) — change the definition of a foreign server

[ALTER STATISTICS](https://www.postgresql.org/docs/15/sql-alterstatistics.html) — change the definition of an extended statistics object

[ALTER SUBSCRIPTION](https://www.postgresql.org/docs/15/sql-altersubscription.html) — change the definition of a subscription

[ALTER SYSTEM](https://www.postgresql.org/docs/15/sql-altersystem.html) — change a server configuration parameter

[ALTER TABLE](https://www.postgresql.org/docs/15/sql-altertable.html) — change the definition of a table

[ALTER TABLESPACE](https://www.postgresql.org/docs/15/sql-altertablespace.html) — change the definition of a tablespace

[ALTER TEXT SEARCH CONFIGURATION](https://www.postgresql.org/docs/15/sql-altertsconfig.html) — change the definition of a text search configuration

[ALTER TEXT SEARCH DICTIONARY](https://www.postgresql.org/docs/15/sql-altertsdictionary.html) — change the definition of a text search dictionary

[ALTER TEXT SEARCH PARSER](https://www.postgresql.org/docs/15/sql-altertsparser.html) — change the definition of a text search parser

[ALTER TEXT SEARCH TEMPLATE](https://www.postgresql.org/docs/15/sql-altertstemplate.html) — change the definition of a text search template

[ALTER TRIGGER](https://www.postgresql.org/docs/15/sql-altertrigger.html) — change the definition of a trigger

[ALTER TYPE](https://www.postgresql.org/docs/15/sql-altertype.html) — change the definition of a type

[ALTER USER](https://www.postgresql.org/docs/15/sql-alteruser.html) — change a database role

[ALTER USER MAPPING](https://www.postgresql.org/docs/15/sql-alterusermapping.html) — change the definition of a user mapping

[ALTER VIEW](https://www.postgresql.org/docs/15/sql-alterview.html) — change the definition of a view

[ANALYZE](https://www.postgresql.org/docs/15/sql-analyze.html) — collect statistics about a database

[BEGIN](https://www.postgresql.org/docs/15/sql-begin.html) — start a transaction block

[CALL](https://www.postgresql.org/docs/15/sql-call.html) — invoke a procedure

[CHECKPOINT](https://www.postgresql.org/docs/15/sql-checkpoint.html) — force a write-ahead log checkpoint

[CLOSE](https://www.postgresql.org/docs/15/sql-close.html) — close a cursor

[CLUSTER](https://www.postgresql.org/docs/15/sql-cluster.html) — cluster a table according to an index

[COMMENT](https://www.postgresql.org/docs/15/sql-comment.html) — define or change the comment of an object

[COMMIT](https://www.postgresql.org/docs/15/sql-commit.html) — commit the current transaction

[COMMIT PREPARED](https://www.postgresql.org/docs/15/sql-commit-prepared.html) — commit a transaction that was earlier prepared for two-phase commit

[COPY](https://www.postgresql.org/docs/15/sql-copy.html) — copy data between a file and a table

[CREATE ACCESS METHOD](https://www.postgresql.org/docs/15/sql-create-access-method.html) — define a new access method

[CREATE AGGREGATE](https://www.postgresql.org/docs/15/sql-createaggregate.html) — define a new aggregate function

[CREATE CAST](https://www.postgresql.org/docs/15/sql-createcast.html) — define a new cast

[CREATE COLLATION](https://www.postgresql.org/docs/15/sql-createcollation.html) — define a new collation

[CREATE CONVERSION](https://www.postgresql.org/docs/15/sql-createconversion.html) — define a new encoding conversion

[CREATE DATABASE](https://www.postgresql.org/docs/15/sql-createdatabase.html) — create a new database

[CREATE DOMAIN](https://www.postgresql.org/docs/15/sql-createdomain.html) — define a new domain

[CREATE EVENT TRIGGER](https://www.postgresql.org/docs/15/sql-createeventtrigger.html) — define a new event trigger

[CREATE EXTENSION](https://www.postgresql.org/docs/15/sql-createextension.html) — install an extension

[CREATE FOREIGN DATA WRAPPER](https://www.postgresql.org/docs/15/sql-createforeigndatawrapper.html) — define a new foreign-data wrapper

[CREATE FOREIGN TABLE](https://www.postgresql.org/docs/15/sql-createforeigntable.html) — define a new foreign table

[CREATE FUNCTION](https://www.postgresql.org/docs/15/sql-createfunction.html) — define a new function

[CREATE GROUP](https://www.postgresql.org/docs/15/sql-creategroup.html) — define a new database role

[CREATE INDEX](https://www.postgresql.org/docs/15/sql-createindex.html) — define a new index

[CREATE LANGUAGE](https://www.postgresql.org/docs/15/sql-createlanguage.html) — define a new procedural language

[CREATE MATERIALIZED VIEW](https://www.postgresql.org/docs/15/sql-creatematerializedview.html) — define a new materialized view

[CREATE OPERATOR](https://www.postgresql.org/docs/15/sql-createoperator.html) — define a new operator

[CREATE OPERATOR CLASS](https://www.postgresql.org/docs/15/sql-createopclass.html) — define a new operator class

[CREATE OPERATOR FAMILY](https://www.postgresql.org/docs/15/sql-createopfamily.html) — define a new operator family

[CREATE POLICY](https://www.postgresql.org/docs/15/sql-createpolicy.html) — define a new row-level security policy for a table

[CREATE PROCEDURE](https://www.postgresql.org/docs/15/sql-createprocedure.html) — define a new procedure

[CREATE PUBLICATION](https://www.postgresql.org/docs/15/sql-createpublication.html) — define a new publication

[CREATE ROLE](https://www.postgresql.org/docs/15/sql-createrole.html) — define a new database role

[CREATE RULE](https://www.postgresql.org/docs/15/sql-createrule.html) — define a new rewrite rule

[CREATE SCHEMA](https://www.postgresql.org/docs/15/sql-createschema.html) — define a new schema

[CREATE SEQUENCE](https://www.postgresql.org/docs/15/sql-createsequence.html) — define a new sequence generator

[CREATE SERVER](https://www.postgresql.org/docs/15/sql-createserver.html) — define a new foreign server

[CREATE STATISTICS](https://www.postgresql.org/docs/15/sql-createstatistics.html) — define extended statistics

[CREATE SUBSCRIPTION](https://www.postgresql.org/docs/15/sql-createsubscription.html) — define a new subscription

[CREATE TABLE](https://www.postgresql.org/docs/15/sql-createtable.html) — define a new table

[CREATE TABLE AS](https://www.postgresql.org/docs/15/sql-createtableas.html) — define a new table from the results of a query

[CREATE TABLESPACE](https://www.postgresql.org/docs/15/sql-createtablespace.html) — define a new tablespace

[CREATE TEXT SEARCH CONFIGURATION](https://www.postgresql.org/docs/15/sql-createtsconfig.html) — define a new text search configuration

[CREATE TEXT SEARCH DICTIONARY](https://www.postgresql.org/docs/15/sql-createtsdictionary.html) — define a new text search dictionary

[CREATE TEXT SEARCH PARSER](https://www.postgresql.org/docs/15/sql-createtsparser.html) — define a new text search parser

[CREATE TEXT SEARCH TEMPLATE](https://www.postgresql.org/docs/15/sql-createtstemplate.html) — define a new text search template

[CREATE TRANSFORM](https://www.postgresql.org/docs/15/sql-createtransform.html) — define a new transform

[CREATE TRIGGER](https://www.postgresql.org/docs/15/sql-createtrigger.html) — define a new trigger

[CREATE TYPE](https://www.postgresql.org/docs/15/sql-createtype.html) — define a new data type

[CREATE USER](https://www.postgresql.org/docs/15/sql-createuser.html) — define a new database role

[CREATE USER MAPPING](https://www.postgresql.org/docs/15/sql-createusermapping.html) — define a new mapping of a user to a foreign server

[CREATE VIEW](https://www.postgresql.org/docs/15/sql-createview.html) — define a new view

[DEALLOCATE](https://www.postgresql.org/docs/15/sql-deallocate.html) — deallocate a prepared statement

[DECLARE](https://www.postgresql.org/docs/15/sql-declare.html) — define a cursor

[DELETE](https://www.postgresql.org/docs/15/sql-delete.html) — delete rows of a table

[DISCARD](https://www.postgresql.org/docs/15/sql-discard.html) — discard session state

[DO](https://www.postgresql.org/docs/15/sql-do.html) — execute an anonymous code block

[DROP ACCESS METHOD](https://www.postgresql.org/docs/15/sql-drop-access-method.html) — remove an access method

[DROP AGGREGATE](https://www.postgresql.org/docs/15/sql-dropaggregate.html) — remove an aggregate function

[DROP CAST](https://www.postgresql.org/docs/15/sql-dropcast.html) — remove a cast

[DROP COLLATION](https://www.postgresql.org/docs/15/sql-dropcollation.html) — remove a collation

[DROP CONVERSION](https://www.postgresql.org/docs/15/sql-dropconversion.html) — remove a conversion

[DROP DATABASE](https://www.postgresql.org/docs/15/sql-dropdatabase.html) — remove a database

[DROP DOMAIN](https://www.postgresql.org/docs/15/sql-dropdomain.html) — remove a domain

[DROP EVENT TRIGGER](https://www.postgresql.org/docs/15/sql-dropeventtrigger.html) — remove an event trigger

[DROP EXTENSION](https://www.postgresql.org/docs/15/sql-dropextension.html) — remove an extension

[DROP FOREIGN DATA WRAPPER](https://www.postgresql.org/docs/15/sql-dropforeigndatawrapper.html) — remove a foreign-data wrapper

[DROP FOREIGN TABLE](https://www.postgresql.org/docs/15/sql-dropforeigntable.html) — remove a foreign table

[DROP FUNCTION](https://www.postgresql.org/docs/15/sql-dropfunction.html) — remove a function

[DROP GROUP](https://www.postgresql.org/docs/15/sql-dropgroup.html) — remove a database role

[DROP INDEX](https://www.postgresql.org/docs/15/sql-dropindex.html) — remove an index

[DROP LANGUAGE](https://www.postgresql.org/docs/15/sql-droplanguage.html) — remove a procedural language

[DROP MATERIALIZED VIEW](https://www.postgresql.org/docs/15/sql-dropmaterializedview.html) — remove a materialized view

[DROP OPERATOR](https://www.postgresql.org/docs/15/sql-dropoperator.html) — remove an operator

[DROP OPERATOR CLASS](https://www.postgresql.org/docs/15/sql-dropopclass.html) — remove an operator class

[DROP OPERATOR FAMILY](https://www.postgresql.org/docs/15/sql-dropopfamily.html) — remove an operator family

[DROP OWNED](https://www.postgresql.org/docs/15/sql-drop-owned.html) — remove database objects owned by a database role

[DROP POLICY](https://www.postgresql.org/docs/15/sql-droppolicy.html) — remove a row-level security policy from a table

[DROP PROCEDURE](https://www.postgresql.org/docs/15/sql-dropprocedure.html) — remove a procedure

[DROP PUBLICATION](https://www.postgresql.org/docs/15/sql-droppublication.html) — remove a publication

[DROP ROLE](https://www.postgresql.org/docs/15/sql-droprole.html) — remove a database role

[DROP ROUTINE](https://www.postgresql.org/docs/15/sql-droproutine.html) — remove a routine

[DROP RULE](https://www.postgresql.org/docs/15/sql-droprule.html) — remove a rewrite rule

[DROP SCHEMA](https://www.postgresql.org/docs/15/sql-dropschema.html) — remove a schema

[DROP SEQUENCE](https://www.postgresql.org/docs/15/sql-dropsequence.html) — remove a sequence

[DROP SERVER](https://www.postgresql.org/docs/15/sql-dropserver.html) — remove a foreign server descriptor

[DROP STATISTICS](https://www.postgresql.org/docs/15/sql-dropstatistics.html) — remove extended statistics

[DROP SUBSCRIPTION](https://www.postgresql.org/docs/15/sql-dropsubscription.html) — remove a subscription

[DROP TABLE](https://www.postgresql.org/docs/15/sql-droptable.html) — remove a table

[DROP TABLESPACE](https://www.postgresql.org/docs/15/sql-droptablespace.html) — remove a tablespace

[DROP TEXT SEARCH CONFIGURATION](https://www.postgresql.org/docs/15/sql-droptsconfig.html) — remove a text search configuration

[DROP TEXT SEARCH DICTIONARY](https://www.postgresql.org/docs/15/sql-droptsdictionary.html) — remove a text search dictionary

[DROP TEXT SEARCH PARSER](https://www.postgresql.org/docs/15/sql-droptsparser.html) — remove a text search parser

[DROP TEXT SEARCH TEMPLATE](https://www.postgresql.org/docs/15/sql-droptstemplate.html) — remove a text search template

[DROP TRANSFORM](https://www.postgresql.org/docs/15/sql-droptransform.html) — remove a transform

[DROP TRIGGER](https://www.postgresql.org/docs/15/sql-droptrigger.html) — remove a trigger

[DROP TYPE](https://www.postgresql.org/docs/15/sql-droptype.html) — remove a data type

[DROP USER](https://www.postgresql.org/docs/15/sql-dropuser.html) — remove a database role

[DROP USER MAPPING](https://www.postgresql.org/docs/15/sql-dropusermapping.html) — remove a user mapping for a foreign server

[DROP VIEW](https://www.postgresql.org/docs/15/sql-dropview.html) — remove a view

[END](https://www.postgresql.org/docs/15/sql-end.html) — commit the current transaction

[EXECUTE](https://www.postgresql.org/docs/15/sql-execute.html) — execute a prepared statement

[EXPLAIN](https://www.postgresql.org/docs/15/sql-explain.html) — show the execution plan of a statement

[FETCH](https://www.postgresql.org/docs/15/sql-fetch.html) — retrieve rows from a query using a cursor

[GRANT](https://www.postgresql.org/docs/15/sql-grant.html) — define access privileges

[IMPORT FOREIGN SCHEMA](https://www.postgresql.org/docs/15/sql-importforeignschema.html) — import table definitions from a foreign server

[INSERT](https://www.postgresql.org/docs/15/sql-insert.html) — create new rows in a table

[LISTEN](https://www.postgresql.org/docs/15/sql-listen.html) — listen for a notification

[LOAD](https://www.postgresql.org/docs/15/sql-load.html) — load a shared library file

[LOCK](https://www.postgresql.org/docs/15/sql-lock.html) — lock a table

[MERGE](https://www.postgresql.org/docs/15/sql-merge.html) — conditionally insert, update, or delete rows of a table

[MOVE](https://www.postgresql.org/docs/15/sql-move.html) — position a cursor

[NOTIFY](https://www.postgresql.org/docs/15/sql-notify.html) — generate a notification

[PREPARE](https://www.postgresql.org/docs/15/sql-prepare.html) — prepare a statement for execution

[PREPARE TRANSACTION](https://www.postgresql.org/docs/15/sql-prepare-transaction.html) — prepare the current transaction for two-phase commit

[REASSIGN OWNED](https://www.postgresql.org/docs/15/sql-reassign-owned.html) — change the ownership of database objects owned by a database role

[REFRESH MATERIALIZED VIEW](https://www.postgresql.org/docs/15/sql-refreshmaterializedview.html) — replace the contents of a materialized view

[REINDEX](https://www.postgresql.org/docs/15/sql-reindex.html) — rebuild indexes

[RELEASE SAVEPOINT](https://www.postgresql.org/docs/15/sql-release-savepoint.html) — destroy a previously defined savepoint

[RESET](https://www.postgresql.org/docs/15/sql-reset.html) — restore the value of a run-time parameter to the default value

[REVOKE](https://www.postgresql.org/docs/15/sql-revoke.html) — remove access privileges

[ROLLBACK](https://www.postgresql.org/docs/15/sql-rollback.html) — abort the current transaction

[ROLLBACK PREPARED](https://www.postgresql.org/docs/15/sql-rollback-prepared.html) — cancel a transaction that was earlier prepared for two-phase commit

[ROLLBACK TO SAVEPOINT](https://www.postgresql.org/docs/15/sql-rollback-to.html) — roll back to a savepoint

[SAVEPOINT](https://www.postgresql.org/docs/15/sql-savepoint.html) — define a new savepoint within the current transaction

[SECURITY LABEL](https://www.postgresql.org/docs/15/sql-security-label.html) — define or change a security label applied to an object

[SELECT](https://www.postgresql.org/docs/15/sql-select.html) — retrieve rows from a table or view

[SELECT INTO](https://www.postgresql.org/docs/15/sql-selectinto.html) — define a new table from the results of a query

[SET](https://www.postgresql.org/docs/15/sql-set.html) — change a run-time parameter

[SET CONSTRAINTS](https://www.postgresql.org/docs/15/sql-set-constraints.html) — set constraint check timing for the current transaction

[SET ROLE](https://www.postgresql.org/docs/15/sql-set-role.html) — set the current user identifier of the current session

[SET SESSION AUTHORIZATION](https://www.postgresql.org/docs/15/sql-set-session-authorization.html) — set the session user identifier and the current user identifier of the current session

[SET TRANSACTION](https://www.postgresql.org/docs/15/sql-set-transaction.html) — set the characteristics of the current transaction

[SHOW](https://www.postgresql.org/docs/15/sql-show.html) — show the value of a run-time parameter

[START TRANSACTION](https://www.postgresql.org/docs/15/sql-start-transaction.html) — start a transaction block

[TRUNCATE](https://www.postgresql.org/docs/15/sql-truncate.html) — empty a table or set of tables

[UNLISTEN](https://www.postgresql.org/docs/15/sql-unlisten.html) — stop listening for a notification

[UPDATE](https://www.postgresql.org/docs/15/sql-update.html) — update rows of a table

[VACUUM](https://www.postgresql.org/docs/15/sql-vacuum.html) — garbage-collect and optionally analyze a database

[VALUES](https://www.postgresql.org/docs/15/sql-values.html) — compute a set of rows

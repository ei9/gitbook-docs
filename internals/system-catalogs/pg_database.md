# 51.15 pg\_database

目錄 pg\_database 儲存有關資料庫一些可用的訊息。資料庫是使用 [CREATE DATABASE](../../reference/sql-commands/create-database.md) 命令建立的。關於某些參數的含義的詳細訊息，請參閱[第 22 章](../../server-administration/managing-databases/)。

與大多數系統目錄不同，pg\_database 在叢集的所有資料庫之間共享：每個叢集只有一個 pg\_database 副本，而不是每個資料庫一個副本。

**Table 51.15. `pg_database` 欄位**

| 名稱              | 型別          | 參閱                                              | 說明                                                                                                                              |
| --------------- | ----------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `oid`           | `oid`       |                                                 | 資料列識別指標（隱藏屬性；必須明確選擇）                                                                                                            |
| `datname`       | `name`      |                                                 | 資料庫名稱                                                                                                                           |
| `datdba`        | `oid`       | [`pg_authid`](pg\_authid.md).oid                | 資料庫的擁有者，通常是建立它的使用者                                                                                                              |
| `encoding`      | `int4`      |                                                 | 此資料庫的字元編碼（pg\_encoding\_to\_char（）可將此數字轉換為編碼名稱）                                                                                 |
| `datcollate`    | `name`      |                                                 | 這個資料庫的 LC\_COLLATE                                                                                                              |
| `datctype`      | `name`      |                                                 | 這個資料庫的 LC\_CTYPE                                                                                                                |
| `datistemplate` | `bool`      |                                                 | 如果為 true，則該資料庫可以由具有 CREATEDB 權限的任何使用者複製；如果為 false，則只有超級使用者或資料庫的擁有者才能複製它。                                                        |
| `datallowconn`  | `bool`      |                                                 | 如果為 false，則沒有人可以連線到該資料庫。這用於保護 template0 資料庫免遭更改。                                                                                |
| `datconnlimit`  | `int4`      |                                                 | 設定可以對此資料庫執行的最大同時連線數。-1 意味著沒有限制。                                                                                                 |
| `datlastsysoid` | `oid`       |                                                 | 資料庫中的最後一個系統 OID；特別適用於 pg\_dump                                                                                                  |
| `datfrozenxid`  | `xid`       |                                                 | 在這個事務 ID 之前在此資料庫中的所有事務 ID，已被替換為永久（「 frozen」）。這用於追踪是否需要清理資料庫以防止事務 ID 重覆或允許縮減 pg\_xact。它是每個資料表 pg\_class.relfrozenxid 的最小值。       |
| `datminmxid`    | `xid`       |                                                 | 此資料庫中的所有 multixact ID 已被替換為該資料庫中的事務 ID。這用於追踪資料庫是否需要清理，以防止 multixact ID 重覆或允許縮減 pg\_multixact。它是每個資料表 pg\_class.relminmxid 的最小值。 |
| `dattablespace` | `oid`       | [`pg_tablespace`](51.54.-pg\_tablespace.md).oid | 資料庫預設的資料表空間。在此資料庫中，pg\_class.reltablespace 為零的所有資料表都將儲存在此資料表空間中；特別是所有非共享系統目錄都將在那裡。                                              |
| `datacl`        | `aclitem[]` |                                                 | 存取權限；詳情請參閱 [GRANT](../../reference/sql-commands/grant.md) 和 [REVOKE](../../reference/sql-commands/revoke.md)                    |

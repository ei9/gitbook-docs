# 27.2. 日誌轉送備用伺服器 Log-Shipping Standby Servers

持續性歸檔可用於建構高可用性（HA）的叢集配置，其中一個或多個備用伺服器準備好在主伺服器發生故障時接管操作。此功能被廣泛稱為熱備份（warm standby）或日誌轉送(Log-Shipping)。

伺服器們是人為的相依，由主伺服器和備用伺服器協同工作以提供此功能。主伺服器以持續性歸檔模式運行，而每個備用伺服器以連續恢復模式運行，從主伺服器讀取 WAL 檔案。毌須更改資料庫的資料表即可啟用此功能，因此與其他一些複寫解決方案相比，它可以提供較低的管理成本。此配置對主伺服器的效能影響也相對較低。

直接將 WAL 記錄從一個資料庫伺服器移動到另一個資料庫伺服器通常被稱為日誌轉送。PostgreSQL 透過一次傳輸 WAL 記錄一個檔案（WAL 段落）來實現基於檔案的日誌轉送。WAL 檔案（16MB）可以在任何距離上輕鬆便宜地運輸，無論是相鄰系統，同一站點的另一個系統，還是地球另一端的其他系統。此技術所需的頻寬依主伺服器的事務速率而變化。基於記錄的日誌傳送更精細，並且通過網路連連逐步更改 WAL（請參閱[第 26.2.5 節](log-shipping-standby-servers.md#26-2-5-streaming-replication)）。

應該注意的是，日誌輸送是非同步的，即 WAL 記錄在事務提交之後被傳送。因此，如果主伺服器遭受災難性故障，則存在資料遺失的可能性；尚未提交的交易將會失去。基於檔案的日誌轉送中的資料遺失的大小可以透過使用 archive\_timeout 參數來限制，該參數可以設定低至數秒鐘。然而，這種低的設定將大大增加檔案傳送所需的頻寬。 串流複寫（參閱[第 26.2.5 節](log-shipping-standby-servers.md#26-2-5-streaming-replication)）允許更小的資料遺失大小。

回復的效率很高，一旦備用轉為主要，備用資料庫通常只需要幾分鐘即可完全可用。因此，這稱為熱備用配置，可提供高可用性。從歸檔的基本備份和回溯還原伺服器將花費相當長的時間，因此該技術僅提供災難恢復的解決方案，而不是高可用性。備用伺服器也可用於唯讀查詢，在這種情況下，它稱為熱備份伺服器。有關更多訊息，請參閱[第 26.5 節](hot-standby.md)。

## 26.2.1. 規畫

建立主伺服器和備用伺服器通常是好的規畫，使它們可以盡可能相似，至少從資料庫伺服器的角度來看。特別是，與資料表空間關聯的路徑名稱將在未修改的情況下傳遞。因此，如果使用此功能，主伺服器和備用伺服器必須具有相同的資料表空間的安裝路徑。請記住，如果在主伺服器上執行 [CREATE TABLESPACE](../../reference/sql-commands/create-tablespace.md)，則必須在執行命令之前在主伺服器和所有備用伺服器上建立所需的所有新安裝點。硬體不需要完全相同，但經驗上，維護兩個相同的系統會比在應用系統的生命週期內維護兩個不同的系統更容易。不過在硬體架構則必須相同 - 例如，從 32 位元到 64 位元系統的搭配則無法運作。

一般來說，無法在不同主要 PostgreSQL 版本的伺服器之間進行日誌傳送。PostgreSQL 全球開發團隊的原則是不要在次要版本升級期間更改磁碟格式，因此在主伺服器和備用伺服器上使用不同的次要版本可能會成功執行。 但是，並沒有保證正式支持，建議您盡可能將主伺服器和備用伺服器保持在同一版本。更新到新的次要版本時，最安全的策略是先更新備用伺服器 - 新的次要版本更有可能從先前的次要版本讀取 WAL 檔案，反過來則不一定。

## 26.2.2. 備用伺服器作業

在備用模式下，伺服器連續套用從主要伺服器所接收的 WAL。備用伺服器可以透過 TCP 連線（串流複寫）從 WAL 歸檔（請參閱 [restore\_command](../server-configuration/write-ahead-log.md#19-5-4-archive-recovery)）。備用伺服器也會嘗試恢復在備用集群的 pg\_wal 目錄中能找到的任何 WAL。這通常發生在伺服器重新啟動之後，當備用資料庫再次重新執行在重新啟動之前從主服務器串流傳輸的 WAL 時，您也可以隨時手動將檔案複製到 pg\_wal 以重新執行它們。

在啟動時，備用資料庫首先恢復存檔路徑中的所有可用的 WAL，然後呼叫 restore\_command。一旦達到 WAL 可用的尾端並且 restore\_command 失敗，它就會嘗試恢復 pg\_wal 目錄中可用的任何WAL。如果失敗，並且已啟用串流複寫，則備用資料庫會嘗試連到主伺服器，並從 archive 或 pg\_wal 中找到的最後一個有效記錄開始串流傳輸 WAL。 如果失敗或未啟用串流複寫，或者稍後中斷連線，則備用資料庫將返回步驟 1 並嘗試再次從存檔中還原交易。pg\_wal 和串流複寫的重試循環一直持續到伺服器停止或觸發故障轉移為止。

退出備用模式，當執行 pg\_ctl promote 或找到觸發器檔案（trigger\_file）時，伺服器將切換到正常操作。在故障轉移之前，將恢復存檔或 pg\_wal 中立即可用的 WAL，但不會嘗試連線到主要伺服器。

## 26.2.3. Preparing the Master for Standby Servers

Set up continuous archiving on the primary to an archive directory accessible from the standby, as described in [Section 25.3](https://www.postgresql.org/docs/10/static/continuous-archiving.html). The archive location should be accessible from the standby even when the master is down, i.e. it should reside on the standby server itself or another trusted server, not on the master server.

If you want to use streaming replication, set up authentication on the primary server to allow replication connections from the standby server(s); that is, create a role and provide a suitable entry or entries in `pg_hba.conf` with the database field set to `replication`. Also ensure `max_wal_senders` is set to a sufficiently large value in the configuration file of the primary server. If replication slots will be used, ensure that `max_replication_slots` is set sufficiently high as well.

Take a base backup as described in [Section 25.3.2](https://www.postgresql.org/docs/10/static/continuous-archiving.html#BACKUP-BASE-BACKUP) to bootstrap the standby server.

## 26.2.4. Setting Up a Standby Server

To set up the standby server, restore the base backup taken from primary server (see [Section 25.3.4](https://www.postgresql.org/docs/10/static/continuous-archiving.html#BACKUP-PITR-RECOVERY)). Create a recovery command file `recovery.conf` in the standby's cluster data directory, and turn on `standby_mode`. Set `restore_command` to a simple command to copy files from the WAL archive. If you plan to have multiple standby servers for high availability purposes, set `recovery_target_timeline` to `latest`, to make the standby server follow the timeline change that occurs at failover to another standby.

### Note

Do not use pg\_standby or similar tools with the built-in standby mode described here. `restore_command` should return immediately if the file does not exist; the server will retry the command again if necessary. See [Section 26.4](https://www.postgresql.org/docs/10/static/log-shipping-alternative.html) for using tools like pg\_standby.

If you want to use streaming replication, fill in `primary_conninfo` with a libpq connection string, including the host name (or IP address) and any additional details needed to connect to the primary server. If the primary needs a password for authentication, the password needs to be specified in `primary_conninfo` as well.

If you're setting up the standby server for high availability purposes, set up WAL archiving, connections and authentication like the primary server, because the standby server will work as a primary server after failover.

If you're using a WAL archive, its size can be minimized using the [archive\_cleanup\_command](https://www.postgresql.org/docs/10/static/archive-recovery-settings.html#ARCHIVE-CLEANUP-COMMAND) parameter to remove files that are no longer required by the standby server. The pg\_archivecleanup utility is designed specifically to be used with `archive_cleanup_command` in typical single-standby configurations, see [pg\_archivecleanup](https://www.postgresql.org/docs/10/static/pgarchivecleanup.html). Note however, that if you're using the archive for backup purposes, you need to retain files needed to recover from at least the latest base backup, even if they're no longer needed by the standby.

A simple example of a `recovery.conf` is:

```
standby_mode = 'on'
primary_conninfo = 'host=192.168.1.50 port=5432 user=foo password=foopass'
restore_command = 'cp /path/to/archive/%f %p'
archive_cleanup_command = 'pg_archivecleanup /path/to/archive %r'
```

You can have any number of standby servers, but if you use streaming replication, make sure you set `max_wal_senders` high enough in the primary to allow them to be connected simultaneously.

## 26.2.5. Streaming Replication

Streaming replication allows a standby server to stay more up-to-date than is possible with file-based log shipping. The standby connects to the primary, which streams WAL records to the standby as they're generated, without waiting for the WAL file to be filled.

Streaming replication is asynchronous by default (see [Section 26.2.8](https://www.postgresql.org/docs/10/static/warm-standby.html#SYNCHRONOUS-REPLICATION)), in which case there is a small delay between committing a transaction in the primary and the changes becoming visible in the standby. This delay is however much smaller than with file-based log shipping, typically under one second assuming the standby is powerful enough to keep up with the load. With streaming replication, `archive_timeout` is not required to reduce the data loss window.

If you use streaming replication without file-based continuous archiving, the server might recycle old WAL segments before the standby has received them. If this occurs, the standby will need to be reinitialized from a new base backup. You can avoid this by setting `wal_keep_segments` to a value large enough to ensure that WAL segments are not recycled too early, or by configuring a replication slot for the standby. If you set up a WAL archive that's accessible from the standby, these solutions are not required, since the standby can always use the archive to catch up provided it retains enough segments.

To use streaming replication, set up a file-based log-shipping standby server as described in [Section 26.2](https://www.postgresql.org/docs/10/static/warm-standby.html). The step that turns a file-based log-shipping standby into streaming replication standby is setting `primary_conninfo` setting in the `recovery.conf` file to point to the primary server. Set [listen\_addresses](https://www.postgresql.org/docs/10/static/runtime-config-connection.html#GUC-LISTEN-ADDRESSES) and authentication options (see `pg_hba.conf`) on the primary so that the standby server can connect to the `replication` pseudo-database on the primary server (see [Section 26.2.5.1](https://www.postgresql.org/docs/10/static/warm-standby.html#STREAMING-REPLICATION-AUTHENTICATION)).

On systems that support the keepalive socket option, setting [tcp\_keepalives\_idle](https://www.postgresql.org/docs/10/static/runtime-config-connection.html#GUC-TCP-KEEPALIVES-IDLE), [tcp\_keepalives\_interval](https://www.postgresql.org/docs/10/static/runtime-config-connection.html#GUC-TCP-KEEPALIVES-INTERVAL) and [tcp\_keepalives\_count](https://www.postgresql.org/docs/10/static/runtime-config-connection.html#GUC-TCP-KEEPALIVES-COUNT) helps the primary promptly notice a broken connection.

Set the maximum number of concurrent connections from the standby servers (see [max\_wal\_senders](https://www.postgresql.org/docs/10/static/runtime-config-replication.html#GUC-MAX-WAL-SENDERS) for details).

When the standby is started and `primary_conninfo` is set correctly, the standby will connect to the primary after replaying all WAL files available in the archive. If the connection is established successfully, you will see a walreceiver process in the standby, and a corresponding walsender process in the primary.

### **26.2.5.1. Authentication**

It is very important that the access privileges for replication be set up so that only trusted users can read the WAL stream, because it is easy to extract privileged information from it. Standby servers must authenticate to the primary as a superuser or an account that has the `REPLICATION` privilege. It is recommended to create a dedicated user account with `REPLICATION` and `LOGIN` privileges for replication. While `REPLICATION` privilege gives very high permissions, it does not allow the user to modify any data on the primary system, which the `SUPERUSER` privilege does.

Client authentication for replication is controlled by a `pg_hba.conf` record specifying `replication` in the _`database`_ field. For example, if the standby is running on host IP `192.168.1.100`and the account name for replication is `foo`, the administrator can add the following line to the `pg_hba.conf` file on the primary:

```
# Allow the user "foo" from host 192.168.1.100 to connect to the primary
# as a replication standby if the user's password is correctly supplied.
#
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    replication     foo             192.168.1.100/32        md5
```

The host name and port number of the primary, connection user name, and password are specified in the `recovery.conf` file. The password can also be set in the `~/.pgpass` file on the standby (specify `replication` in the _`database`_ field). For example, if the primary is running on host IP `192.168.1.50`, port `5432`, the account name for replication is `foo`, and the password is `foopass`, the administrator can add the following line to the `recovery.conf` file on the standby:

```
# The standby connects to the primary that is running on host 192.168.1.50
# and port 5432 as the user "foo" whose password is "foopass".
primary_conninfo = 'host=192.168.1.50 port=5432 user=foo password=foopass'
```

### **26.2.5.2. Monitoring**

An important health indicator of streaming replication is the amount of WAL records generated in the primary, but not yet applied in the standby. You can calculate this lag by comparing the current WAL write location on the primary with the last WAL location received by the standby. These locations can be retrieved using `pg_current_wal_lsn` on the primary and `pg_last_wal_receive_lsn` on the standby, respectively (see [Table 9.79](https://www.postgresql.org/docs/10/static/functions-admin.html#FUNCTIONS-ADMIN-BACKUP-TABLE) and [Table 9.80](https://www.postgresql.org/docs/10/static/functions-admin.html#FUNCTIONS-RECOVERY-INFO-TABLE) for details). The last WAL receive location in the standby is also displayed in the process status of the WAL receiver process, displayed using the `ps` command (see [Section 28.1](https://www.postgresql.org/docs/10/static/monitoring-ps.html) for details).

You can retrieve a list of WAL sender processes via the [`pg_stat_replication`](https://www.postgresql.org/docs/10/static/monitoring-stats.html#PG-STAT-REPLICATION-VIEW) view. Large differences between `pg_current_wal_lsn` and the view's `sent_lsn` field might indicate that the master server is under heavy load, while differences between `sent_lsn` and `pg_last_wal_receive_lsn` on the standby might indicate network delay, or that the standby is under heavy load.

## 26.2.6. Replication Slots

Replication slots provide an automated way to ensure that the master does not remove WAL segments until they have been received by all standbys, and that the master does not remove rows which could cause a [recovery conflict](https://www.postgresql.org/docs/13/hot-standby.html#HOT-STANDBY-CONFLICT) even when the standby is disconnected.

In lieu of using replication slots, it is possible to prevent the removal of old WAL segments using [wal\_keep\_size](https://www.postgresql.org/docs/13/runtime-config-replication.html#GUC-WAL-KEEP-SIZE), or by storing the segments in an archive using [archive\_command](https://www.postgresql.org/docs/13/runtime-config-wal.html#GUC-ARCHIVE-COMMAND). However, these methods often result in retaining more WAL segments than required, whereas replication slots retain only the number of segments known to be needed. On the other hand, replication slots can retain so many WAL segments that they fill up the space allocated for `pg_wal`; [max\_slot\_wal\_keep\_size](https://www.postgresql.org/docs/13/runtime-config-replication.html#GUC-MAX-SLOT-WAL-KEEP-SIZE) limits the size of WAL files retained by replication slots.

Similarly, [hot\_standby\_feedback](https://www.postgresql.org/docs/13/runtime-config-replication.html#GUC-HOT-STANDBY-FEEDBACK) and [vacuum\_defer\_cleanup\_age](https://www.postgresql.org/docs/13/runtime-config-replication.html#GUC-VACUUM-DEFER-CLEANUP-AGE) provide protection against relevant rows being removed by vacuum, but the former provides no protection during any time period when the standby is not connected, and the latter often needs to be set to a high value to provide adequate protection. Replication slots overcome these disadvantages.

### **26.2.6.1. Querying and manipulating replication slots**

Each replication slot has a name, which can contain lower-case letters, numbers, and the underscore character.

Existing replication slots and their state can be seen in the [`pg_replication_slots`](https://www.postgresql.org/docs/13/view-pg-replication-slots.html) view.

Slots can be created and dropped either via the streaming replication protocol (see [Section 52.4](https://www.postgresql.org/docs/13/protocol-replication.html)) or via SQL functions (see [Section 9.27.6](https://www.postgresql.org/docs/13/functions-admin.html#FUNCTIONS-REPLICATION)).

### **26.2.6.2. Configuration Example**

You can create a replication slot like this:

```
postgres=# SELECT * FROM pg_create_physical_replication_slot('node_a_slot');
  slot_name  | lsn
-------------+-----
 node_a_slot |

postgres=# SELECT slot_name, slot_type, active FROM pg_replication_slots;
  slot_name  | slot_type | active 
-------------+-----------+--------
 node_a_slot | physical  | f
(1 row)
```

To configure the standby to use this slot, `primary_slot_name` should be configured on the standby. Here is a simple example:

```
primary_conninfo = 'host=192.168.1.50 port=5432 user=foo password=foopass'
primary_slot_name = 'node_a_slot'
```

## 26.2.7. Cascading Replication

The cascading replication feature allows a standby server to accept replication connections and stream WAL records to other standbys, acting as a relay. This can be used to reduce the number of direct connections to the master and also to minimize inter-site bandwidth overheads.

A standby acting as both a receiver and a sender is known as a cascading standby. Standbys that are more directly connected to the master are known as upstream servers, while those standby servers further away are downstream servers. Cascading replication does not place limits on the number or arrangement of downstream servers, though each standby connects to only one upstream server which eventually links to a single master/primary server.

A cascading standby sends not only WAL records received from the master but also those restored from the archive. So even if the replication connection in some upstream connection is terminated, streaming replication continues downstream for as long as new WAL records are available.

Cascading replication is currently asynchronous. Synchronous replication (see [Section 26.2.8](https://www.postgresql.org/docs/13/warm-standby.html#SYNCHRONOUS-REPLICATION)) settings have no effect on cascading replication at present.

Hot Standby feedback propagates upstream, whatever the cascaded arrangement.

If an upstream standby server is promoted to become new master, downstream servers will continue to stream from the new master if `recovery_target_timeline` is set to `'latest'` (the default).

To use cascading replication, set up the cascading standby so that it can accept replication connections (that is, set [max\_wal\_senders](https://www.postgresql.org/docs/13/runtime-config-replication.html#GUC-MAX-WAL-SENDERS) and [hot\_standby](https://www.postgresql.org/docs/13/runtime-config-replication.html#GUC-HOT-STANDBY), and configure [host-based authentication](https://www.postgresql.org/docs/13/auth-pg-hba-conf.html)). You will also need to set `primary_conninfo` in the downstream standby to point to the cascading standby.

## 26.2.8. Synchronous Replication

PostgreSQL streaming replication is asynchronous by default. If the primary server crashes then some transactions that were committed may not have been replicated to the standby server, causing data loss. The amount of data loss is proportional to the replication delay at the time of failover.

Synchronous replication offers the ability to confirm that all changes made by a transaction have been transferred to one or more synchronous standby servers. This extends that standard level of durability offered by a transaction commit. This level of protection is referred to as 2-safe replication in computer science theory, and group-1-safe (group-safe and 1-safe) when `synchronous_commit` is set to `remote_write`.

When requesting synchronous replication, each commit of a write transaction will wait until confirmation is received that the commit has been written to the write-ahead log on disk of both the primary and standby server. The only possibility that data can be lost is if both the primary and the standby suffer crashes at the same time. This can provide a much higher level of durability, though only if the sysadmin is cautious about the placement and management of the two servers. Waiting for confirmation increases the user's confidence that the changes will not be lost in the event of server crashes but it also necessarily increases the response time for the requesting transaction. The minimum wait time is the round-trip time between primary to standby.

Read only transactions and transaction rollbacks need not wait for replies from standby servers. Subtransaction commits do not wait for responses from standby servers, only top-level commits. Long running actions such as data loading or index building do not wait until the very final commit message. All two-phase commit actions require commit waits, including both prepare and commit.

A synchronous standby can be a physical replication standby or a logical replication subscriber. It can also be any other physical or logical WAL replication stream consumer that knows how to send the appropriate feedback messages. Besides the built-in physical and logical replication systems, this includes special programs such as `pg_receivewal` and `pg_recvlogical`as well as some third-party replication systems and custom programs. Check the respective documentation for details on synchronous replication support.

### **26.2.8.1. Basic Configuration**

Once streaming replication has been configured, configuring synchronous replication requires only one additional configuration step: [synchronous\_standby\_names](https://www.postgresql.org/docs/10/static/runtime-config-replication.html#GUC-SYNCHRONOUS-STANDBY-NAMES) must be set to a non-empty value. `synchronous_commit` must also be set to `on`, but since this is the default value, typically no change is required. (See [Section 19.5.1](https://www.postgresql.org/docs/10/static/runtime-config-wal.html#RUNTIME-CONFIG-WAL-SETTINGS) and [Section 19.6.2](https://www.postgresql.org/docs/10/static/runtime-config-replication.html#RUNTIME-CONFIG-REPLICATION-MASTER).) This configuration will cause each commit to wait for confirmation that the standby has written the commit record to durable storage. `synchronous_commit` can be set by individual users, so it can be configured in the configuration file, for particular users or databases, or dynamically by applications, in order to control the durability guarantee on a per-transaction basis.

After a commit record has been written to disk on the primary, the WAL record is then sent to the standby. The standby sends reply messages each time a new batch of WAL data is written to disk, unless `wal_receiver_status_interval` is set to zero on the standby. In the case that `synchronous_commit` is set to `remote_apply`, the standby sends reply messages when the commit record is replayed, making the transaction visible. If the standby is chosen as a synchronous standby, according to the setting of `synchronous_standby_names` on the primary, the reply messages from that standby will be considered along with those from other synchronous standbys to decide when to release transactions waiting for confirmation that the commit record has been received. These parameters allow the administrator to specify which standby servers should be synchronous standbys. Note that the configuration of synchronous replication is mainly on the master. Named standbys must be directly connected to the master; the master knows nothing about downstream standby servers using cascaded replication.

Setting `synchronous_commit` to `remote_write` will cause each commit to wait for confirmation that the standby has received the commit record and written it out to its own operating system, but not for the data to be flushed to disk on the standby. This setting provides a weaker guarantee of durability than `on` does: the standby could lose the data in the event of an operating system crash, though not a PostgreSQL crash. However, it's a useful setting in practice because it can decrease the response time for the transaction. Data loss could only occur if both the primary and the standby crash and the database of the primary gets corrupted at the same time.

Setting `synchronous_commit` to `remote_apply` will cause each commit to wait until the current synchronous standbys report that they have replayed the transaction, making it visible to user queries. In simple cases, this allows for load balancing with causal consistency.

Users will stop waiting if a fast shutdown is requested. However, as when using asynchronous replication, the server will not fully shutdown until all outstanding WAL records are transferred to the currently connected standby servers.

### **26.2.8.2. Multiple Synchronous Standbys**

Synchronous replication supports one or more synchronous standby servers; transactions will wait until all the standby servers which are considered as synchronous confirm receipt of their data. The number of synchronous standbys that transactions must wait for replies from is specified in `synchronous_standby_names`. This parameter also specifies a list of standby names and the method (`FIRST` and `ANY`) to choose synchronous standbys from the listed ones.

The method `FIRST` specifies a priority-based synchronous replication and makes transaction commits wait until their WAL records are replicated to the requested number of synchronous standbys chosen based on their priorities. The standbys whose names appear earlier in the list are given higher priority and will be considered as synchronous. Other standby servers appearing later in this list represent potential synchronous standbys. If any of the current synchronous standbys disconnects for whatever reason, it will be replaced immediately with the next-highest-priority standby.

An example of `synchronous_standby_names` for a priority-based multiple synchronous standbys is:

```
synchronous_standby_names = 'FIRST 2 (s1, s2, s3)'
```

In this example, if four standby servers `s1`, `s2`, `s3` and `s4` are running, the two standbys `s1` and `s2` will be chosen as synchronous standbys because their names appear early in the list of standby names. `s3` is a potential synchronous standby and will take over the role of synchronous standby when either of `s1` or `s2` fails. `s4` is an asynchronous standby since its name is not in the list.

The method `ANY` specifies a quorum-based synchronous replication and makes transaction commits wait until their WAL records are replicated to _at least_ the requested number of synchronous standbys in the list.

An example of `synchronous_standby_names` for a quorum-based multiple synchronous standbys is:

```
synchronous_standby_names = 'ANY 2 (s1, s2, s3)'
```

In this example, if four standby servers `s1`, `s2`, `s3` and `s4` are running, transaction commits will wait for replies from at least any two standbys of `s1`, `s2` and `s3`. `s4` is an asynchronous standby since its name is not in the list.

The synchronous states of standby servers can be viewed using the `pg_stat_replication` view.

### **26.2.8.3. Planning for Performance**

Synchronous replication usually requires carefully planned and placed standby servers to ensure applications perform acceptably. Waiting doesn't utilize system resources, but transaction locks continue to be held until the transfer is confirmed. As a result, incautious use of synchronous replication will reduce performance for database applications because of increased response times and higher contention.

PostgreSQL allows the application developer to specify the durability level required via replication. This can be specified for the system overall, though it can also be specified for specific users or connections, or even individual transactions.

For example, an application workload might consist of: 10% of changes are important customer details, while 90% of changes are less important data that the business can more easily survive if it is lost, such as chat messages between users.

With synchronous replication options specified at the application level (on the primary) we can offer synchronous replication for the most important changes, without slowing down the bulk of the total workload. Application level options are an important and practical tool for allowing the benefits of synchronous replication for high performance applications.

You should consider that the network bandwidth must be higher than the rate of generation of WAL data.

### **26.2.8.4. Planning for High Availability**

`synchronous_standby_names` specifies the number and names of synchronous standbys that transaction commits made when `synchronous_commit` is set to `on`, `remote_apply` or `remote_write` will wait for responses from. Such transaction commits may never be completed if any one of synchronous standbys should crash.

The best solution for high availability is to ensure you keep as many synchronous standbys as requested. This can be achieved by naming multiple potential synchronous standbys using `synchronous_standby_names`.

In a priority-based synchronous replication, the standbys whose names appear earlier in the list will be used as synchronous standbys. Standbys listed after these will take over the role of synchronous standby if one of current ones should fail.

In a quorum-based synchronous replication, all the standbys appearing in the list will be used as candidates for synchronous standbys. Even if one of them should fail, the other standbys will keep performing the role of candidates of synchronous standby.

When a standby first attaches to the primary, it will not yet be properly synchronized. This is described as `catchup` mode. Once the lag between standby and primary reaches zero for the first time we move to real-time `streaming` state. The catch-up duration may be long immediately after the standby has been created. If the standby is shut down, then the catch-up period will increase according to the length of time the standby has been down. The standby is only able to become a synchronous standby once it has reached `streaming`state. This state can be viewed using the `pg_stat_replication` view.

If primary restarts while commits are waiting for acknowledgement, those waiting transactions will be marked fully committed once the primary database recovers. There is no way to be certain that all standbys have received all outstanding WAL data at time of the crash of the primary. Some transactions may not show as committed on the standby, even though they show as committed on the primary. The guarantee we offer is that the application will not receive explicit acknowledgement of the successful commit of a transaction until the WAL data is known to be safely received by all the synchronous standbys.

If you really cannot keep as many synchronous standbys as requested then you should decrease the number of synchronous standbys that transaction commits must wait for responses from in `synchronous_standby_names` (or disable it) and reload the configuration file on the primary server.

If the primary is isolated from remaining standby servers you should fail over to the best candidate of those other remaining standby servers.

If you need to re-create a standby server while transactions are waiting, make sure that the commands pg\_start\_backup() and pg\_stop\_backup() are run in a session with `synchronous_commit` = `off`, otherwise those requests will wait forever for the standby to appear.

## 26.2.9. Continuous archiving in standby

When continuous WAL archiving is used in a standby, there are two different scenarios: the WAL archive can be shared between the primary and the standby, or the standby can have its own WAL archive. When the standby has its own WAL archive, set `archive_mode` to `always`, and the standby will call the archive command for every WAL segment it receives, whether it's by restoring from the archive or by streaming replication. The shared archive can be handled similarly, but the `archive_command` must test if the file being archived exists already, and if the existing file has identical contents. This requires more care in the `archive_command`, as it must be careful to not overwrite an existing file with different contents, but return success if the exactly same file is archived twice. And all that must be done free of race conditions, if two servers attempt to archive the same file at the same time.

If `archive_mode` is set to `on`, the archiver is not enabled during recovery or standby mode. If the standby server is promoted, it will start archiving after the promotion, but will not archive any WAL it did not generate itself. To get a complete series of WAL files in the archive, you must ensure that all WAL is archived, before it reaches the standby. This is inherently true with file-based log shipping, as the standby can only restore files that are found in the archive, but not if streaming replication is enabled. When a server is not in recovery mode, there is no difference between `on` and `always` modes.

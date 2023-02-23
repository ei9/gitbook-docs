# 20.4. 資源配置

## 20.4.1. 記憶體

`shared_buffers` (`integer`)

設定資料庫伺服器用於共享記憶體緩衝區的大小。預設值通常為 128 MB，但如果您的核心設定不支援（在 initdb 期間確定），則可能會更少。此設定必須至少為128 KB。（非預設值的 BLCKSZ 會改變最小值。）但是，通常需要高於最小值的設定才能獲得良好的性能。此參數只能在伺服器啟動時設定。

如果您擁有 1GB 或更多記憶體的專用資料庫伺服器，shared\_buffers 的合理起始值是系統記憶體的 25％。有些工作負載甚至可以為 shared\_buffers 設定更大的值，但由於PostgreSQL 依賴於作業系統緩衝區，因此，把 shared\_buffers 分配 40％ 以上的記憶體大小不太可能比少量分配更好。shared\_buffers 較大設定通常需要 max\_wal\_size 相對應的增加，以便分散在較長時間內寫入大量新資料或變更資料的過程。

在 RAM 小於 1GB 的系統上，更小比例是合適的，以便為作業系統留下足夠的空間。

`huge_pages` (`enum`)

啟用/停用大型記憶體頁面。有效值為 try（預設值），on 和 off。

目前，僅在 Linux 上支援此功能。設定為 try 時，在其他系統上會忽略該設定。

大型頁面的使用會使得頁面管理表更小，記憶體管理花費的 CPU 時間更少，從而提高了效能。有關更多詳細訊息，請參閱[第 18.4.5 節](../server-setup-and-operation/managing-kernel-resources.md#18-4-5-linux-huge-pages)。

設定 huge\_pages 後，伺服器將嘗試使用大型頁面，但如果失敗則回退到使用正常分配。如果為 on，則若無法使用大型頁面將使伺服器無法啟動。 off 時，則不會使用大型頁面。

`temp_buffers` (`integer`)

設定每個資料庫連線使用的最大臨時緩衝區大小。這些是僅用於存取臨時資料表的連線本地緩衝區。預設值為 8MB。可以在單個連線中變更設定，但只能在連線中首次使用臨時資料表之前更改；後續嘗試更改該值將不會對該連線產生任何影響。

連線將根據需要分配臨時緩衝區，直到 temp\_buffers 的上限。實際上不需要很多臨時緩衝區的連線中設定較大值的成本只是 temp\_buffers 中每個增量的緩衝區描述指標，或大約 64 個位元組。但是，如果實際使用緩衝區，則會消耗額外的 8192 位元組（或者通常為 BLCKSZ 個位元組）。

`max_prepared_transactions` (`integer`)

設定可同時處於「prepared」狀態的最大交易事務數量（請參閱 [PREPARE TRANSACTION](../../reference/sql-commands/prepare-transaction.md)）。將此參數設定為零（這是預設值）的話，會停用預備交易的功能。此參數只能在伺服器啟動時設定。

如果您不打算使用預備交易事務，則應將此參數設定為零以防止意外建立預備的交易事務。如果您正在使用預備的交易事務，那麼您可能希望 max\_prepared\_transactions 至少與 [max\_connections](connections-and-authentication.md#19-3-1-ding) 一樣大，以便每個連線都可以至少有一個準備好的預備交易事務。

運行備用伺服器時，必須將此參數設定為與主服務器上相同或更高的值。 否則，查詢將不被允許在備用伺服器中。

`work_mem` (`integer`)

指定寫入暫存檔之前內部排序操作和雜湊表使用的記憶體大小。此值預設為 4 MB。請注意，對於複雜的查詢，可能會同時執行多個排序或雜湊作業；在開始將資料寫入暫存檔之前，每個操作都將被允許盡可能使用記憶體。此外，多個連線可以同時進行這些操作。因此，所使用的總記憶體量可能是 work\_mem 值的許多倍；決定值時必須牢記此一事實。排序操作用於 ORDER BY，DISTINCT 和 merge JOIN。雜湊表用於 hash JOIN，hash aggregation 和 IN 子查詢處理。

`hash_mem_multiplier` (`floating point`)

Used to compute the maximum amount of memory that hash-based operations can use. The final limit is determined by multiplying `work_mem` by `hash_mem_multiplier`. The default value is 1.0, which makes hash-based operations subject to the same simple `work_mem` maximum as sort-based operations.

Consider increasing `hash_mem_multiplier` in environments where spilling by query operations is a regular occurrence, especially when simply increasing `work_mem` results in memory pressure (memory pressure typically takes the form of intermittent out of memory errors). A setting of 1.5 or 2.0 may be effective with mixed workloads. Higher settings in the range of 2.0 - 8.0 or more may be effective in environments where `work_mem` has already been increased to 40MB or more.

`maintenance_work_mem` (`integer`)

指定維護操作要使用的最大記憶體大小，例如 VACUUM，CREATE INDEX 和ALTER TABLE ADD FOREIGN KEY。預設為 64 MB。由於資料庫連線一次只能執行其中一個操作，不會有多個同時運行，因此將此值設定為遠大於 work\_mem 是安全的。較大的設定可能會提高清理和恢復資料庫回復的效能。

請注意，當 autovacuum 運行時，最多可以分配 [autovacuum\_max\_workers](automatic-vacuuming.md) 倍的記憶體，因此請注意不要將預設值設定得太高。透過單獨設定 [autovacuum\_work\_mem](resource-consumption.md#19-4-1) 來控制它會有幫助。

`autovacuum_work_mem` (`integer`)

指定每個 autovacuum 工作程序使用的最大記憶體。它預設為 -1，表示應該使用 [maintenance\_work\_mem](resource-consumption.md#19-4-1) 的值。以其他方式執行時，此設定對 VACUUM 的行為沒有影響。

`logical_decoding_work_mem` (`integer`)

Specifies the maximum amount of memory to be used by logical decoding, before some of the decoded changes are written to local disk. This limits the amount of memory used by logical streaming replication connections. It defaults to 64 megabytes (`64MB`). Since each replication connection only uses a single buffer of this size, and an installation normally doesn't have many such connections concurrently (as limited by `max_wal_senders`), it's safe to set this value significantly higher than `work_mem`, reducing the amount of decoded changes written to disk.

`max_stack_depth` (`integer`)

指定伺服器工作堆疊的最大安全深度。此參數的理想設定是核心強制執行的實際堆疊大小限制（由 ulimit -s 或其他等效設定），減去 1 MB 左右的安全範圍。需要安全額度，因為在伺服器的每個程序中都不會檢查堆疊深度，而是僅在關鍵的潛在遞迴程序（例如表示式求值）中檢查。預設設定是 2 MB，這是保守地小，不太可能冒崩潰的風險。但是，它可能太小而無法執行複雜的功能。只有超級使用者才能變更此設定。

將 max\_stack\_depth 設定為高於實際核心限制將意味著失控的遞迴函數可能導致單個後端程序崩潰。在 PostgreSQL 可以確定核心限制的平台上，伺服器不允許將此變數設定為不安全的值。但是，並非所有平台都有提供資訊，因此建議在選擇值時要小心。

`shared_memory_type` (`enum`)

Specifies the shared memory implementation that the server should use for the main shared memory region that holds PostgreSQL's shared buffers and other shared data. Possible values are `mmap` (for anonymous shared memory allocated using `mmap`), `sysv` (for System V shared memory allocated via `shmget`) and `windows` (for Windows shared memory). Not all values are supported on all platforms; the first supported option is the default for that platform. The use of the `sysv` option, which is not the default on any platform, is generally discouraged because it typically requires non-default kernel settings to allow for large allocations (see [Section 18.4.1](https://www.postgresql.org/docs/13/kernel-resources.html#SYSVIPC)).

`dynamic_shared_memory_type` (`enum`)

指定伺服器應使用的動態共享記憶體方法。可能的值是 posix（使用 shm\_open 分配的 POSIX 共享記憶體），sysv（透過 shmget 分配的 System V 共享記憶體），windows（Windows 共享記憶體），mmap（使用儲存在資料目錄中的記憶體映射檔案來模擬共享記憶體） ），沒有（停用此功能）。並非所有平台都支援所有值；第一個受支援的選項是該平台的預設選項。通常不鼓勵使用 mmap 選項，這在任何平台上都不是預設選項，因為作業系統可能會將修改後的頁面重複寫回磁碟，從而增加系統 I/O 負載；但是，當 pg\_dynshmem 目錄儲存在 RAM 磁碟上或其他共享記憶體裝置不可用時，它可能對除錯很有用。

## 20.4.2. 磁碟

`temp_file_limit` (`integer`)

指定程序可用於暫存檔的最大磁碟空間大小，例如排序和雜湊暫存檔，或持有游標的檔案。試圖超過此限制的交易將被取消。此值以 KB 為單位指定，-1（預設值）表示無限制。只有超級使用者可以變更改此設定。

此設定限制了給予 PostgreSQL 程序使用的所有暫存檔在任何時刻能使用的總空間。應該注意的是，用於臨時資料表的磁碟空間與在查詢執行過程中使用的暫存檔不同，並不會計入此限制。

## 20.4.3. 核心資源配置

`max_files_per_process` (`integer`)

設定每個伺服器子程序允許的同時最大開啓的檔案數。預設值是 1000 個檔案。如果核心可以確保每個程序的安全限制，則不必擔心此設定。但是在某些平台上（特別是大多數 BSD 系統），如果許多程序都嘗試開啓那麼多檔案，核心將允許單個程序打開比系統實際支援的更多的檔案。如果您發現自己看到“Too many open files”失敗，請嘗試減少此設定。此參數只能在伺服器啟動時設定。

## 20.4.4. 成本考量的 Vacuum 延遲

在執行 [VACUUM](../../reference/sql-commands/vacuum.md) 和 [ANALYZE](../../reference/sql-commands/analyze.md) 指令期間，系統會維護一個內部計數器，用於追踪執行的各種 I/O 操作的估計成本。當累計成本達到極限（由 vacuum\_cost\_limit 指定）時，執行操作的過程將在 sleep\_cost\_delay 指定的短時間內休眠。然後它將重置計數器並繼續執行。

此功能的目的是允許管理員減少這些指令對同時間資料庫活動的 I/O 影響。在許多情況下，像 VACUUM 和 ANALYZE 這樣的維護指令很快完成就不重要；但是，這些指令又通常非常重要，不會嚴重干擾系統執行其他資料庫操作的能力。基於成本的清理延遲為管理員提供了實現這一目標的途徑。

對於手動發出的 VACUUM 指令，預設情況下會停用此功能。要啟用它，請將 vacuum\_cost\_delay 變數設定為非零值。

`vacuum_cost_delay` (`integer`)

超出成本限制時程序將休眠的時間長度（以毫秒為單位）。預設值為零，這會停用成本考量的清理延遲功能。正值可實現成本考量的清理。請注意，在許多系統上，睡眠延遲的有效分辨率為 10 毫秒；將 vacuum\_cost\_delay 設定為不是 10 的倍數的值可能與將其設定為 10 的下一個更高倍數具有相同的結果。

當使用成本考量的資料庫清理時，vacuum\_cost\_delay 的適當值通常非常小，可能是 10 或 20 毫秒。調整清理的資源消耗最好透過變更其他清理成本參數來完成。

`vacuum_cost_page_hit` (`integer`)

清除共享緩衝區中找到的緩衝區估計成本。它表示鎖定緩衝池，查詢共享雜湊表和掃描頁面內容的成本。預設值為 1。

`vacuum_cost_page_miss` (`integer`)

清除必須從磁碟讀取的緩衝區的估計成本。這表示鎖定緩衝池，查詢共享雜湊表，從磁碟讀取所需塊並掃描其內容的成本。預設值為 10。

`vacuum_cost_page_dirty` (`integer`)

清理修改先前清理的區塊時産生的估計成本。它表示將已修改區塊再次更新到磁碟所需的額外 I/O。預設值為 20。

`vacuum_cost_limit` (`integer`)

累積成本將導致清理程序進入睡眠狀態。預設值為 200。

### 注意

某些操作可能會持有關鍵的鎖定，因此應盡快完成。在此類操作期間不會發生成本考量的清理延遲。因此，成本可能會遠遠高於指定的限制。為了避免在這種情況下無意義的長延遲，實際延遲計算為 vacuum\_cost\_delay \_\_\* cumulative\_balance / vacuum\_cost\_limit，最大為 vacuum\_cost\_delay \_\*\_ 4。

## 20.4.5. 背景寫入程序

有一個單獨的伺服器程序稱為背景寫入程序，其功能是發起「dirty」（新的或修改的）共享緩衝區的寫入。 它會寫入共享緩衝區，因此處理使用者查詢的伺服器程序很少或永遠不需要等待寫入的發生。但是，背景寫入程序確實導致 I/O 負載的整體的淨增加，因為雖然每個檢查點間隔可能只會寫一次 repeatedly-dirtied 頁面，但背景寫入程序可能會發起多次寫入，因為它在同一時間間隔內被變更了。本小節中討論的參數可用於調整適於本地需求的行為。

`bgwriter_delay` (`integer`)

指定背景寫入程序的輪詢之間的延遲。在每一次輪詢中，寫入程序發出一些 dirty 緩衝區的寫入（可透過以下參數控制）。然後它睡眠 bgwriter\_delay 毫秒，再重複。但是，當緩衝池中沒有 dirty 緩衝區時，無論 bgwriter\_delay 如何，它都會進入更長的睡眠狀態。預設值為 200 毫秒。請注意，在許多系統上，睡眠延遲的有效分辨率為 10 毫秒；將 bgwriter\_delay 設定為不是 10 的倍數可能與將其設定為 10 的下一個更高倍數具有相同的結果。此參數只能在 postgresql.conf 檔案或伺服器命令列中設定。

`bgwriter_lru_maxpages` (`integer`)

在每一次輪詢中，背景寫入程序將寫入多個緩衝區。將此值設定為零將停用背景寫入。（請注意，由單獨的專用輔助程序管理的檢查點不受影響。）預設值為 100 個緩衝區。此參數只能在 postgresql.conf 檔案或伺服器命令列中設定。

`bgwriter_lru_multiplier` (`floating point`)

每次輪詢寫入的 dirty 緩衝區數量取決於最近幾輪中伺服器程序所需的新緩衝區數。將最近的平均需求乘以 bgwriter\_lru\_multiplier，得出下一輪期間所需緩衝區數量的估計值。寫入 dirty 緩衝區，直到有許多乾淨，可再利用的緩衝區可用。（但是，每輪不會寫入超過 bgwriter\_lru\_maxpages 的緩衝區。）因此，1.0 的設定表示準確寫出預測需要的緩衝區數量的「Just in time」策略。較大的值為需求中的峰值提供了一些緩衝，而較小的值有意地使寫入由伺服器程序完成。預設值為 2.0。 此參數只能在 postgresql.conf 檔案或伺服器命令列中設定。

`bgwriter_flush_after` (`integer`)

只要背景寫入程序寫入了超過 bgwriter\_flush\_after 個位元組，就會嘗試強制作業系統向底層儲存系統發出這些寫入操作。這樣做會限制核心頁面緩衝區中的 dirty 資料量，減少在檢查點結束時發出 fsync 時停止的可能性，或者作業系統在背景以較大批次寫回資料的可能性。通常這會導致事務延遲大大減少，但也有一些情況，特別是工作負載大於 shared\_buffers，但小於作業系統的頁面緩衝，其效能可能會降低。 此設定可能對某些平台沒有影響。有效範圍介於 0（停用強制寫回）和2MB之間。Linux 上的預設值為 512kB，其他地方為 0。（如果 BLCKSZ 不是8kB，則預設值和最大值會按比例縮放。）此參數只能在 postgresql.conf 檔案或匼服器命令列中設定。

bgwriter\_lru\_maxpages 和 bgwriter\_lru\_multiplier 設定較小值可以減少背景寫入程序造成的額外 I/O 負載，但使伺服器程序更有可能必須為自己發出寫入要求，可能造成交互查詢的延遟。

## 20.4.6. 非同步作業

#### `effective_io_concurrency` (`integer`)

設定 PostgreSQL 期望可以同時執行的磁碟 I/O 操作數。提高此值將增加任何單個 PostgreSQL 連線嘗試同時啟動的 I/O 操作數。允許的範圍是 1 到 1000，或者為零以停用非同步 I/O 要求的使用。目前，此設定僅影響 bitmap heap 掃描。

對於磁碟機而言，此設定一個很好的起點是包含用於資料庫的 RAID 0 分散或 RAID 1 鏡像的單獨磁碟數量。（對於 RAID 5，不應計算奇偶校驗磁碟。）但是，如果資料庫通常忙於在同時連線中發出多個查詢，則較低的值可能足以使磁碟陣列保持忙碌狀態。高於保持磁碟繁忙所需的值只會導致額外的 CPU 開銷。SSD和其他基於內存的儲存通常可以處理許多同時要求，因此最佳值可能是數百個。

非同步 I/O 取決於某些作業系統缺乏的有效 posix\_fadvise 函數。如果該功能不存在，則將此參數設定為零以外的任何值將導致錯誤。而在某些作業系統（例如，Solaris）上，此功能存在但實際上並沒有做任何事情。

在受支援的系統上預設值為 1，否則為 0。透過設定同名的 tablespace 參數，可以為特定資料表空間中的資料表覆寫此值（請參閱 [ALTER TABLESPACE](../../reference/sql-commands/alter-tablespace.md)）。

#### `maintenance_io_concurrency` (`integer`)

Similar to `effective_io_concurrency`, but used for maintenance work that is done on behalf of many client sessions.

The default is 10 on supported systems, otherwise 0. This value can be overridden for tables in a particular tablespace by setting the tablespace parameter of the same name (see [ALTER TABLESPACE](https://www.postgresql.org/docs/13/sql-altertablespace.html)).

#### `max_worker_processes` (`integer`)

設定系統可以支援的最大背景程序數量。此參數只能在伺服器啟動時設定。預定值為 8。

執行備用伺服器時，必須將此參數設定為與主伺服器上相同或更高的值。否則，將不允許在備用伺服器中進行查詢。

變更此值時，請考慮同步調整 max\_parallel\_workers 和 max\_parallel\_workers\_per\_gather。

#### `max_parallel_workers_per_gather` (`integer`)

設定單個 Gather 或 Gather Merge 節點可以啟動的最大工作程序數量。同時工作程序取自 max\_worker\_processes 建立的程序池，由 max\_parallel\_workers 限制。請注意，請求的工作程序數量在執行時可能實際上不可用。如果發生這種情況，計劃將以比預期更少的工作程序運行，這可能是低效能的。預設值為 2。將此值設定為 0 將停用平行查詢執行。

請注意，平行查詢可能比非平行查詢消耗的資源要多得多，因為每個工作程序都是一個完全獨立的程序，與其他使用者連線對系統的影響大致相同。在為此設定選擇值時，以及在配置控制資源利用率的其他設定（例如work\_mem）時，應考慮這一點。 諸如 work\_mem 之類的資源限制被單獨應用於每個工作程序，這意味著所有程序的總利用率可能比通常用於任何單個程序的總利用率高得多。例如，使用 4 個工作程序的平行查詢可能會使用高達 5 倍的 CPU 時間、記憶體、I/O 頻寬等作為根本不使用工作程序的查詢。

有關平行查詢的更多訊息，請參閱[第 15 章](https://github.com/pgsql-tw/gitbook-docs/tree/67cc71691219133f37b9a33df9c691a2dd9c2642/tw/the-sql-language/15.-ping-hang-cha-xun)。

#### `max_parallel_maintenance_workers` (`integer`)

Sets the maximum number of parallel workers that can be started by a single utility command. Currently, the parallel utility commands that support the use of parallel workers are `CREATE INDEX` only when building a B-tree index, and `VACUUM` without `FULL` option. Parallel workers are taken from the pool of processes established by [max\_worker\_processes](https://www.postgresql.org/docs/13/runtime-config-resource.html#GUC-MAX-WORKER-PROCESSES), limited by [max\_parallel\_workers](https://www.postgresql.org/docs/13/runtime-config-resource.html#GUC-MAX-PARALLEL-WORKERS). Note that the requested number of workers may not actually be available at run time. If this occurs, the utility operation will run with fewer workers than expected. The default value is 2. Setting this value to 0 disables the use of parallel workers by utility commands.

Note that parallel utility commands should not consume substantially more memory than equivalent non-parallel operations. This strategy differs from that of parallel query, where resource limits generally apply per worker process. Parallel utility commands treat the resource limit `maintenance_work_mem` as a limit to be applied to the entire utility command, regardless of the number of parallel worker processes. However, parallel utility commands may still consume substantially more CPU resources and I/O bandwidth.

`max_parallel_workers` (`integer`)

設定系統可以支援平行查詢的最大工作程序數量。預設值為 8。增大或減小此值時，請考慮調整 max\_parallel\_workers\_per\_gather。另請注意，此值的設定高於 max\_worker\_processes 將不起作用，因為平行工作程序取自該設定所建立的工作程序池。

`backend_flush_after` (`integer`)

只要一個後端寫入了多個 backend\_flush\_after 字串，就會嘗試強制作業系統向底層儲存發出這些寫入操作。這樣做會限制核心頁面緩衝區中的非同步資料量，減少在檢查點結束時發出 fsync 時暫時停止的可能性，或者作業系統在後端以較大批量寫回資料的可能性。通常這會導致事務延遲大大減少，但也有一些情況，特別是工作負載大於shared\_buffers，但小於作業系統的頁面暫存，其性能可能會降低。此設定可能對某些平台沒有影響。有效範圍介於 0（停用強制寫回）和 2MB 之間。預設值為 0，即沒有強制寫回。（如果 BLCKSZ 不是 8kB，則最大值與其成比例。）

`old_snapshot_threshold` (`integer`)

設定可以使用快照的最短時間，而不會在使用快照時發生快照過舊的錯誤。此參數只能在伺服器啟動時設定。

超過閾值，舊資料可能被清理。這可以幫助防止長時間使用的快照所面臨的資料膨脹。為了防止由於清理快照可能會顯示資料的錯誤結果，當快照早於此閾值時會産生錯誤，並且快照用於讀取自建構快照以來已修改的頁面。

值 -1 將停用此功能，並且是預設值。産品等級的有用值可能從少量幾小時到幾天不等。此設定將被強制為分鐘的顆粒度，並且僅允許小數字（例如 0 或 1 分鐘），因為它們有時可用於測試。雖然允許設定高達 60d，但請注意，在許多工作負載中，可能會在更短的時間範圍內發生極端資料膨脹或事務 ID 重覆。

啟用此功能後，關連末尾釋放的空間無法釋放到作業系統，因為這可能會刪除檢測快照過舊狀態所需的訊息。除非明確要求釋放（例如，使用 VACUUM FULL），否則分配給關連的所有空間仍與該關連相關聯，僅在該關連內重覆使用。

此設定不會嘗試保證在任何特定情況下都會産生錯誤。實際上，如果可以從已完成結果集合的游標産生正確的結果，即使引用資料表中的基礎資料列已被清理，也不會産生錯誤。有些資料表不能安全地儘早清理，因此不會受到此設定的影響，例如系統目錄。對於此類資料表，此設定既不會減少膨脹，也不會在掃描時產生快照過舊的錯誤。

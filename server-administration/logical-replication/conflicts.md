# 31.5. 衝突處理

邏輯複寫的行為與正常的 DML 操作類似，因為即使在訂閱節點上變更了資料，資料也會更新。 如果傳入的資料違反任何限制條件，複寫將會停止。這被稱為衝突。當複寫 UPDATE 或 DELETE 操作時，失去的資料不會產生衝突，而這些操作將會簡單地跳過。

衝突會產生錯誤並停止複寫；它必須由使用者手動解決。有關衝突的詳細訊息可以在使用者的伺服器日誌中找到。

解決方案可以透過變更訂閱戶上的資料來完成，以避免與傳入變更衝突或跳過與現有資料衝突的交易事務。透過呼叫 [pg\_replication\_origin\_advance()](../../the-sql-language/functions-and-operators/system-administration.md#9-26-6-replication-functions) 函數以及與訂閱名稱和位置相對應的 node\_name，可以跳過該交易事務。可以在 [pg\_replication\_origin\_status](../../internals/system-catalogs/51.79.-pg\_replication\_origin\_status.md) 系統檢視表中看到開始的目前位置。

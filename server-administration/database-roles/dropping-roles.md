# 22.4. 移除角色

因為角色可以擁有資料庫物件，並且可以擁有存取其他物件的權限，所以移除角色通常不僅僅是快速 [DROP USER](../../reference/sql-commands/drop-user.md) 的問題。該角色擁有的任何物件必須先被移除或重新分配給其他使用者；而授予角色的任何權限也都必須被撤銷。

物件的所有權可以使用 ALTER 指令一次轉換，例如：

```
ALTER TABLE bobs_table OWNER TO alice;
```

或者，可以使用 [REASSIGN OWNED](../../reference/sql-commands/reassign-owned.md) 指令將要移除角色擁有的所有物件的所有權重新分配給另一個角色。由於 REASSIGN OWNED 無法存取其他資料庫中的物件，因此有必要在包含該角色擁有的物件的每個資料庫中執行它。（請注意，第一個這樣的 REASSIGN OWNED 將改變任何共享的資料庫間物件，即資料庫或資料表空間的所有權，這些資料庫或資料表空間由將被移除的角色所擁有）。

一旦任何有價值的物件已經轉移到新的所有者中，則可以使用 [DROP OWNED](../../reference/sql-commands/drop-owned.md) 指令移弓除待移除角色擁有的任何剩餘物件。同樣，此指令無法存取其他資料庫中的物件，因此有必要在包含該角色擁有的物件的每個資料庫中執行它。此外，DROP OWNED 不會刪除整個資料庫或資料表空間，因此如果角色擁有尚未轉移到新所有者的任何資料庫或資料表空間，則必須手動執行此操作。

DROP OWNED 還負責為不屬於它的物件移除授予目標角色的所有權限。由於 REASSIGN OWNED 不會觸及這些物件，因此通常需要運行 REASSIGN OWNED 和 DROP OWNED（按此順序！）以完全移除要移除的角色的相依關係。

簡而言之，移除已用於擁有物件的角色的最一般的方式是：

```
REASSIGN OWNED BY doomed_role TO successor_role;
DROP OWNED BY doomed_role;
-- repeat the above commands in each database of the cluster
DROP ROLE doomed_role;
```

當並非所有擁有的物件都將被轉移到相同的繼任者使用者時，最好手動處理異常，然後執行上述步驟來清除。

如果在相依物件仍然存在的情況下嘗試 DROP ROLE，則會發出哪些物件需要重新分配或移除的訊息。

# 6.4. 修改並回傳資料

有時在修改資料列的操作過程中取得資料是很方便的。INSERT、UPDATE 和 DELETE 指令都有一個選擇性的RETURNING 子句來支持這個功能。使用 RETURNING 可以避免執行額外的資料庫查詢來收集資料，特別是在難以可靠地識別修改的資料列時尤其有用。

RETURNING 子句允許的語法與 SELECT 指令的輸出列表相同（詳見[第 7.3 節](../queries/select-lists.md)）。它可以包含命令目標資料表的欄位名稱，或者包含使用這些欄位的表示式。常用的簡寫形式是 RETURNING \*，預設是資料表的所有欄位，且相同次序。

在 INSERT 中，可用於 RETURNING 的資料是新增的資料列。這在一般的資料新增中並不是很有用，因為它只會重複用戶端所提供的資料。但如果是計算過的預設值就會非常方便。 例如，當使用串列欄位（[serial](../data-types/numeric-types.md#8-1-4-serial-types)）提供唯一識別時，RETURNING 可以回傳分配給新資料列的 ID：

```
CREATE TABLE users (firstname text, lastname text, id serial primary key);

INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;
```

對於 INSERT ... SELECT，RETURNING 子句也非常有用。

在 UPDATE 中，可用於 RETURNING 的資料是被修改的資料列新內容。例如：

```
UPDATE products SET price = price * 1.10
  WHERE price <= 99.99
  RETURNING name, price AS new_price;
```

在 DELETE 中，可用於 RETURNING 的資料是已刪除資料列的內容。例如：

```
DELETE FROM products
  WHERE obsoletion_date = 'today'
  RETURNING *;
```

如果目標資料表上有觸發函數的話（[第 38 章](../../server-programming/triggers/)），則可用於 RETURNING 的資料是由該觸發函數所修改的資料列。因此，由觸發函數計算檢查欄位是 RETURNING 的另一個常見用法。

# 9.28. 觸發函式

雖然觸發器的許多用途涉及使用者自行定義的觸發器函數，但 PostgreSQL 提供了一些可以直接在使用者定義的觸發器中使用的內建觸發器函數。這些都列在 [Table 9.97](trigger-functions.md#table-9-97-built-in-trigger-functions) 之中。（還有一些是附加的內建觸發器函數，它們用於實現外部鍵和延遲性的索引限制條件。由於使用者並不需要直接使用它們，因此這裡就沒有將它們文件化。）

有關建立觸發器的更多說明，請參閱 [CREATE TRIGGER](../../reference/sql-commands/create-trigger.md)。

#### **Table 9.97. Built-In Trigger Functions**

| <p>Function</p><p>Description</p><p>Example Usage</p>                                                                                                                                                                                                                                                                                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><code>suppress_redundant_updates_trigger</code> ( ) → <code>trigger</code></p><p>跳過不會產生具體的更新行為。 詳情請見下文。</p><p><code>CREATE TRIGGER ... suppress_redundant_updates_trigger()</code></p>                                                                                                                                                                                           |
| <p><code>tsvector_update_trigger</code> ( ) → <code>trigger</code></p><p>從關聯的純文字檔欄位自動更新 tsvector 欄位。要使用的文字搜尋配置以名稱指定為觸發器參數。有關詳細說明，請參閱<a href="../12.-quan-wen-jian-suo/12.4.-yan-shen-gong-neng.md#12-4-3-triggers-for-automatic-updates">第 12.4.3 節</a>。</p><p><code>CREATE TRIGGER ... tsvector_update_trigger(tsvcol, 'pg_catalog.swedish', title, body)</code></p>               |
| <p><code>tsvector_update_trigger_column</code> ( ) → <code>trigger</code></p><p>從關聯的純文字檔欄位自動更新 tsvector 欄位。 要使用的文字搜尋配置取自資料表的 regconfig 欄位。有關詳細說明，請參閱<a href="../12.-quan-wen-jian-suo/12.4.-yan-shen-gong-neng.md#12-4-3-triggers-for-automatic-updates">第 12.4.3 節</a>。</p><p><code>CREATE TRIGGER ... tsvector_update_trigger_column(tsvcol, tsconfigcol, title, body)</code></p> |

當作用於資料列層級的 BEFORE UPDATE 觸發器時，suppress\_redundant\_updates\_trigger 函數將阻止任何不實際變更資料表資料的 UPDATE 行為發生。這會替代掉無論如何都會正常執行的實體資料更新行為，無論是否發生資料變更。（這種行為使 UPDATE 執行得更快，因為不需要重覆進行相關的檢查，並且在某些情況下也很有用。）

理想情況下，您應該避免執行實際上不會變更資料的 UPDATE。多餘的 UPDATE 可能會花費大量不必要的時間，尤其是當有大量索引要更改，並且棄置的資料空間必須時常被清理時。然而，在客戶端程式中檢測這種情況並不是這麼容易，甚至是不可能的，而且撰寫表示式來檢測也容易出錯。另一種方法則是使用 suppress\_redundant\_updates\_trigger，它會跳過不改變資料的 UPDATE。觸發器為每筆資料花費很少但非常重要的時間，因此如果UPDATE 條件的大多數資料確實發生了變化，則使用此觸發器將使更新執行得比平均的情況更慢。

Suppress\_redundant\_updates\_trigger 函數可以像這樣加到資料表中來使用：

```
CREATE TRIGGER z_min_update
BEFORE UPDATE ON tablename
FOR EACH ROW EXECUTE FUNCTION suppress_redundant_updates_trigger();
```

在大多數情況下，對於每一筆資料來說，您需要在最後才觸發此觸發器，以便它不會妨礙可能希望更改該筆資料的其他觸發器。請記住，觸發器是按名稱次序觸發的，因此您應該選擇觸發器名稱在資料表上可能有的任何其他觸發器的名稱之後。（因此在範例中使用「z」的開頭名稱。）

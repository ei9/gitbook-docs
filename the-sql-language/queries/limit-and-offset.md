# 7.6. LIMIT 和 OFFSET

LIMIT 和 OFFSET 允許你只回傳由查詢生成的一部分資料列：

```
SELECT select_list
    FROM table_expression
    [ ORDER BY ... ]
    [ LIMIT { number | ALL } ] [ OFFSET number]
```

如果給了一個限制的數量，那麼只有那個數目的資料列會回傳（如果查詢本身產生較少的資料列，則可能會少一些）。LIMIT ALL 與省略 LIMIT 子句相同，也如同 LIMIT 的參數為 NULL。

OFFSET 指的是在開始回傳資料列之前跳過那麼多少資料列。OFFSET 0 與忽略 OFFSET 子句相同，就像使用 NULL 參數的 OFFSET 一樣。

如果同時出現 OFFSET 和 LIMIT，則在開始計算回傳的LIMIT 資料列之前，先跳過 OFFSET 數量的資料列。

使用 LIMIT 時，運用 ORDER BY 子句將結果資料列限制為唯一順序非常重要。否則，你會得到一個不可預知的查詢資料列的子集。你可能會查詢第十到第二十個資料列，但是第十到第二十個資料列是按什麼順序排列的？次序是未知的，除非你指定 ORDER BY。

查詢最佳化在產生查詢計劃時會將 LIMIT 考慮在內，所以根據你給的 LIMIT 和 OFFSET，你很可能會得到不同的計劃（產生不同的資料列順序）。因此，使用不同的 LIMIT / OFFSET 值來選擇查詢結果的不同子集將導致不一致的結果，除非使用 ORDER BY 強制執行可預測的結果排序。這不是一個錯誤；這是一種事實上的結果，即 SQL 不保證以任何特定順序傳遞查詢的結果，除非使用 ORDER BY 來約束順序。

由 OFFSET 子句跳過的資料列仍然需要在伺服器內計算。因此一個大的 OFFSET 可能是低效率的。

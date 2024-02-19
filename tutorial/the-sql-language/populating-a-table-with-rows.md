# 2.4. 資料列是資料表的組成單位

INSERT 指令被用來將資料以資料列（row）的形式，新增至資料表（table）之中：

```
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

注意，所有的資料型別都有明確的輸入格式。只要不是簡單的數值內容，都必須要以單引號（'）括住，如同在本例中的形式。日期時間型別（date type）的資料內容就比較有彈性，但在這個導覽之中，我們仍然使用較固定的格式來表現。

地理資訊型別（point type）需要有座標組作為輸入，如下所示：

```
INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');
```

到目前為止，語法的使用需要你依照欄位宣告的次序擺放，而另一種語法可以允許你明確地指定資料相對應的欄位：

```
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

你可以將欄位以不同的次序擺放，甚或略去某些欄位，例如，precipitation 欄位（prcp）內容未知：

```
INSERT INTO weather (date, city, temp_hi, temp_lo)
    VALUES ('1994-11-29', 'Hayward', 54, 37);
```

許多開發者會認為，在撰寫習慣上，明確指定欄位是比較好的方式。

請執行下列的指令，你將會擁有後續章節所需要的範例資料。

你可能需要使用 COPY 這個指令從文字檔案來載入大量的資料。這個指令會比 INSERT 要快上許多，因為 COPY 指令的設計就是為了大量資料輸入而產生的。它少了一些彈性，但提供了效率上的最佳表現。使用範例如下所示：

```
COPY weather FROM '/home/user/weather.txt';
```

資料來源的檔案必須存在於後端的伺服器之中，並且可被 PostgreSQL 使用者（postgres）所存取，注意不是用戶端的主機，因為後端伺服器的服務需要直接讀取該檔案。你可以取得更多詳細說明，在 [COPY 指令](../../reference/sql-commands/copy.md)的說明頁面。

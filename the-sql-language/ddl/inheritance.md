# 5.10. 繼承

PostgreSQL 實作了資料表的繼承方式，對於資料庫設計人員來說，將會是很好用的功能。（SQL:1999 之後定義了型別繼承的功能，但和這裡所介紹的方向有許多不同。）

我們直接以一個例子作為開始：假設我們嘗試建立「城市（city）」的資料模型。每一個州（state）都會有許多城市（city），但只會有一個首都（capital）。我們想要很快地可以找到某個州的首都。這件事我們需要建立兩個資料表，一個存放首都，而另一個記載非首都的城市。只是，當我們想要取得的是所有城市，不論是否為首都，似乎會有些麻煩？這時候繼承功能就可以幫助我們解決這個問題。我們可以定義一個資料表 capitals，它是由資料表 cities 繼承而來：

```
CREATE TABLE cities (
    name            text,
    population      float,
    altitude        int     -- in feet
);

CREATE TABLE capitals (
    state           char(2)
) INHERITS (cities);
```

在這個例子中，資料表 capitals 會繼承父資料表 cities 的所有欄位。只是 capitals 會多一個欄位 state，表示它是哪個州的首都。

在 PostgreSQL 裡，一個資料表可以繼承多個資料表，而一個查詢可以引用該資料表裡的所有資料列或在其所屬的資料表的資料列，後者的行為是預設的。舉個例子，下面的查詢可以列出所有海沷在 500 英呎以上的城市名稱，州的首都也包含在內：

```
SELECT name, altitude
    FROM cities
    WHERE altitude > 500;
```

使用 [2.1 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/the-sql-language/21-introduction.md)中的範例資料，將會回傳：

```
   name    | altitude
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
 Madison   |      845
```

換句話說，下面的查詢就會查出非首都且海沷超過 500 英呎以上的城市：

```
SELECT name, altitude
    FROM ONLY cities
    WHERE altitude > 500;

   name    | altitude
-----------+----------
 Las Vegas |     2174
 Mariposa  |     1953
```

這裡「ONLY」關鍵字指的是查詢只需要包含資料表 cities 就好，而不是任何繼承 cities 的資料表都包含在內。我們先前介紹過的指令：SELECT、UPDATE、和 DELETE，都可以使用 ONLY 關鍵字。

你也可以在資料表名稱後面加上「\*」，明確指出繼承的資料表都需要包含在內：

```
SELECT name, altitude
    FROM cities*
    WHERE altitude > 500;
```

注意這個「\*」並不是必要的，因為這個行為本來就是預設的。這個語法用於相容舊的版本，有些版本的預設行為可能不太一樣。

在某些例子裡，也許你會希望知道哪些資料列來自於哪個資料表。有一個系統欄位稱作 tableoid，每一個資料表都會有，而它可告訴你資料列的來源：

```
SELECT c.tableoid, c.name, c.altitude
FROM cities c
WHERE c.altitude > 500;
```

這將會回傳：

```
 tableoid |   name    | altitude
----------+-----------+----------
   139793 | Las Vegas |     2174
   139793 | Mariposa  |     1953
   139798 | Madison   |      845
```

（如果你嘗試重覆執行這個例子，你可能會得到不同的 OID 值。）藉由和資料表 pg\_class 交叉查詢，你可以看到實際的資料表名稱：

```
SELECT p.relname, c.name, c.altitude
FROM cities c, pg_class p
WHERE c.altitude > 500 AND c.tableoid = p.oid;
```

將會回傳：

```
 relname  |   name    | altitude
----------+-----------+----------
 cities   | Las Vegas |     2174
 cities   | Mariposa  |     1953
 capitals | Madison   |      845
```

另一個可以得到相同結果的方式是，使用 regclass 別名型別，這個型別會將 OID 轉換成名稱輸出：

```
SELECT c.tableoid::regclass, c.name, c.altitude
FROM cities c
WHERE c.altitude > 500;
```

在使用 INSERT 或 COPY 指令時，繼承並不會自動轉存資料。在我們的例子中，下面的 INSERT 指令將會失敗：

```
INSERT INTO cities (name, population, altitude, state)
VALUES ('Albany', NULL, NULL, 'NY');
```

我們可能會希望資料以某種方式轉送到資料表 capitals 中，但這不會發生：INSERT 指令永遠只會將資料插入到指定的資料表中。在某些情況下，如果設定了存取規則（[第 40 章](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/v-server-programming/the-rule-system.md)）的話，那有可能做到類似的效果。然而，在這個例子下是沒有辦法執行的，因為資料表 cities 中並沒有一個欄位稱作 state，所以這個指令將會被拒絕執行，如果沒有其他規則被設定的話。

所有限制條件的檢查，還有非空值的限制，都會自動從父資料表繼承下來，除非特別使用 NO INHERIT 子句來設定拋棄繼承。而其他型態的限制條件（唯一性、主鍵、外部鍵）都不會自動繼承。

一個資料表也可以繼承超過一個資料表，也就是說，它會擁有這些資料表全部的欄位，然後再加上自己所宣告的欄位。如果父資料表有相同名稱的欄位的話，或是父資料表和子資料表有同名的欄位，那麼這些欄位會被合併，它們會被合併為一個欄位。合併的時候，他們的資料型別必須要一致，否則會產生錯誤。被繼承的限制條件和無空值的限制也會用類似的方式合併。舉個例子來說，如果要合併的欄位中，任何一個欄位有 not-null 的設定的話，那麼合併後的欄位就會被設定為 not-null。如果有同名的限制條件要被合併，但他們的內容不相同的話，那麼合併也會失敗。

資料表的繼承一般來說是在子資料表建立時進行的，也就是在 [CREATE TABLE](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/create-table.md) 中使用 INHERITES 子句。然而，資料表也可以在 [ALTER TABLE](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/alter-table.md) 中使用 INHERIT 子句來新增新的父資料表。要進行這個動作，新的子資料表必須已經包含所有父資料表的欄位—相同的欄位名稱及資料型別。還有在 ALTER TABLE 時加入 INHERIT 子句來移除某個欄位的繼承。動態地新增或移除繼承欄位通常是在應用分割表格（table partitioning）時特別好用（請參閱 [5.10 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-definition/510-table-partitioning.md)）。

還有一個方便的方式去建立一個相容於之後繼承的資料表，就是在 CREATE TABLE 中使用 LIKE 子句。這個方式會建立一個新的資料表，其欄位和另一個資料表完全相同。如果有任何 CHECK 子句的限制條件的話，就應該在 LIKE 子句中加入 INCLUDING CONSTRAINTS 選項，這樣就會和父資料表完全相容了。

父資料表無法在子資料表仍然存在時被移除。子資料表的欄位和限制條件也不能被移除，如果它們是由其他資料表繼承而來的話。如果你想要移除某個資料表，包含其相關的物件的話，一個簡單的方式就是在移除時加上 CASCADE 選項（請參閱 [5.13 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-definition/513-dependency-tracking.md)）。

[ALTER TABLE](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/alter-table.md) 將會讓欄位型態和限制條件的改變，衍生至繼承它的資料表之中。一樣地，移除某個欄位，如果它有被其他資料表繼承的話，那麼就必須要加上 CASCADE 選項才行。ALTER TABLE 會遵循和 CREATE TABLE 一樣的規則，決定重覆的欄位要合併還是拒絕。

指令的繼承權限是依父資料表的權限。舉個例子，當你存取資料表 cities 時，在 cities 上給予 UPDATE 的權限，同時也隱含了賦予 capitals 更新資料的權限。這考量到這些資料也會出現在父資料表，但如果你沒有特別給予 capitals 權限的話，你還是無法直接存取 capitals。類似的情況也會發生在資料列的安全原則（[5.7 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-definition/57-row-security-policies.md)），在繼承查詢時，同樣是參考父資料表的安全原則。而子資料表額外的安全原則，只在直接查詢該資料表時有效，同時任何父資料表的安全原則會失效。

外部資料表（[5.11 節](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/ii-the-sql-language/data-definition/511-foreign-data.md)）也可以是繼承的一部份，父資料表或子資料表，就如同一般的資料表一樣。只是，如果整個繼承結構中，有任何外部資料不支援的操作的話，那麼整個繼承結構就都不支援。

## 5.9.1. 警告

注意，並非所有的 SQL 指令都可以在繼承結構中執行。一般常用的資料查詢，資料更新，或結構調整（像是 SELECT、UPDATE、DELETE，還有多數 ALTER TABLE 的功能，但不包括 INSERT 或 ALTER TABLE ...... RENAME），基本上預設都是包含子資料表，也支援使用 ONLY 指示字來排除子資料表。如果是資料庫維護或調教的指令，如 REINDEX、VACUUM，一般就只支援特定且實體的資料表，就不會在繼承結構中衍生其他的動作。這些個別指令相關的行為，請參閱 [SQL Commands](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands.md) 內的說明。

繼承功能比較嚴格的限制是索引（包含唯一性索引），還有外部鍵的限制條件，都只能用在單一資料表，而不會衍生至他們的子資料表中。對外部鍵來說，無論引用資料表或是參考資料表的情況都一樣。下面是一些例子說明：

* 如果我們宣告 cities.name 具備唯一性或是主鍵，這不會限制到 capitals 中有重覆的項目。而這些重覆的資料列就會出現在 cities 的查詢結果中。事實上，預設的 capitals 就沒有唯一性的限制，所以就可能有多個資料列記載相同的名稱。你可以在 capitals 中也加入唯一性索引，但這也無法避免 capitals 和 cities 中有重覆的項目。
* 同樣地，如果我們指定 cities.name 以外部鍵的方式，參考另一個資料表，而這個外部鍵也不會衍生到 capitals 中。這種情況你就必須在 capitals 中也以 REFERENCES 設定同樣外部鍵的引用。
* 如果有另一個資料表的欄位設定了 REFERENCES cities(name) 就會允許其他的資料表包含城市名稱，但就沒有首都名稱。在這個情況下，沒有好的解決辦法。

這些缺點可能會在後續的版本中被修正，但在此時此刻，當你需要使用繼承功能讓你的應用設計更好用時，你就必須要同時考慮這些限制。

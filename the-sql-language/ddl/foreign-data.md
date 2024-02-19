# 5.12. 外部資料

PostgreSQL 實作了 SQL/MED 的部份標準，讓你可以存取不在 PostgreSQL 管理下的資料，重點是，你仍然只需要使用 SQL 語法。這樣的資料我們稱作為外部資料。（注意這部份的使用不要和外部鍵搞混了，外部鍵是資料庫內部的一種條件限制。）

外部資料的存取是透過「Foreign data wrapper」（外部資料封裝技術）。外部資料封裝技術是一組函式庫，用於和外部的資料源溝通，它封裝了資料連線和存取資料的細節。有一些外部資料封裝的套件收錄在 contrib 模組之中，參閱附件 F。其他種類的外部封裝套件則由第三方產品提供。如果沒有適合你的資料源的套件的話，你也可以自己寫一個，參閱第 56 章。

要存取外部資料，你需要建立外部服務物件，用它來連結特定的外部資料源，也可以對套件進行一些設定。然後你還需要建立幾個外部資料表，用於定義外部資料的資料結構。外部資料表的使用就如一般的表格一樣，只不過它沒有實際儲存任何資料罷了。當外部資料表被查詢時，PostgreSQL 會透過外部資料封裝套件，從外部資料源取得資料，或者傳送資料到外部，進行更新資料。

存取外部資料可能需要對外部資料源進行認證。這可以利用使用者映對（user mapping）的方法，讓每個 PostgreSQL 使用者在使用部資料表時，可以傳送自己的認證資訊。

進一步的資訊，請參閱 [CREATE FOREIGN DATA WRAPPER](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/create-foreign-data-wrapper.md)、[CREATE SERVER](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/create-server.md)、[CREATE USER MAPPING](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/create-user-mapping.md)、[CREATE FOREIGN TABLE](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/create-foreign-table.md)、[IMPORT FOREIGN SCHEMA ](https://github.com/pgsql-tw/documents/tree/a096b206440e1ac8cdee57e1ae7a74730f0ee146/vi-reference/i-sql-commands/import-foreign-schema.md)等內容。

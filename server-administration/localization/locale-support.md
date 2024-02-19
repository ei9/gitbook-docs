# 24.1. 語系支援

區域設定支援是指某個應用程序，它提供有關字母、排序、數字格式等文化偏好。PostgreSQL 使用伺服器作業系統提供的標準 ISO C 和 POSIX 區域設定。有關其他訊息，請參閱作業系統文件。

## 24.1.1. 綜觀

使用 initdb 建立資料庫叢集時，將自動初始化語言環境支援。initdb 將預設使用其執行環境的語言環境設定初始化資料庫叢集。因此，如果您的作業系統已設定為使用資料庫叢集中所需的語言環境，那麼您毌須進行任何額外操作。如果要使用其他語言環境（或者您不確定系統設定的語言環境），可以透過指定 --locale 選項指示 initdb 確切使用哪個語言環境。例如：

```
initdb --locale=sv_SE
```

Unix 系統的這個範例將語言環境設定為瑞典語（SE）中的瑞典語（sv）。其他可能性可能包括 en\_US（美國英語）和 fr\_CA（加拿大法語）。如果可以將多個字元集用於語言環境，則規範可採用 language\_territory.codeset 形式。例如，fr\_BE.UTF-8 表示比利時（BE）中使用的法語（fr），具有 UTF-8 字元集編碼。

系統上可用的區域設定取決於作業系統供應商提供和安裝的內容。在大多數 Unix 系統上，指令 `locale -a` 將提供可用語言環境的列表。Windows 使用更詳細的區域設定名稱，例如 German\_Germany 或 Swedish\_Sweden.1252，但原則是相同的。

有時，混合來自多個語言環境的規則很有用，例如，使用英語校對規則，但使用西班牙語訊息。為了支援這一點，可以存在一組區域設定子類別，它們僅控制本地化規則的某些方面：

| `LC_COLLATE`  | String sort order    |
| ------------- | -------------------- |
| `LC_CTYPE`    | 字元分類（什麼是字母？它的大寫字母是？） |
| `LC_MESSAGES` | 訊息的語言                |
| `LC_MONETARY` | 格式化貨幣金額              |
| `LC_NUMERIC`  | 格式化數字                |
| `LC_TIME`     | 格式化日期和時間             |

類別名稱轉換為 initdb 選項的名稱，以覆蓋特定類別的區域設定選項。例如，要將語言環境設定為加拿大法語，但使用美國規則格式化貨幣，請使用 `initdb --locale = fr_CA --lc-monetary = en_US`。

如果您希望系統的行為就像它沒有語言環境支援一樣，請使用特殊的語言環境名稱 C 或等效的 POSIX。

建立資料庫時，某些區域設定類別必須固定其值。 您可以對不同的資料庫使用不同的設定，但是一旦建立了資料庫，就無法再為該資料庫更改它們。LC\_COLLATE 和 LC\_CTYPE 是這些類別。它們會影響索引的排序順序，因此必須保持不變，否則文字欄位上的索引會損壞。（但是您可以使用排序規則來緩解此限制，如[第 24.2 節](collation-support.md)中所述。）這些類別的預設值在執行 initdb 時確定，並且在建立新資料庫時使用這些值，除非在 CREATE DATABASE 指令中另行指定。

透過設定與語言環境類別同名的伺服器配置參數，可以隨時更改其他語言環境類別（有關詳細訊息，請參閱[第 20.11.2 節](../server-configuration/client-connection-defaults.md#19-11-2-xi-ge-shi)）。initdb 選擇的值實際上只寫入配置文件 postgresql.conf，以在伺服器啟動時用作預設值。如果從 postgresql.conf 中刪除這些設定，則伺服器將從其執行環境繼承設定。

請注意，服務器的區域設定行為由伺服器看到的環境變數決定，而不是由任何用戶端的環境確定。因此，在啟動伺服器之前，請務必配置正確的區域設定。這樣做的結果是，如果用戶端和伺服器設定在不同的區域設定中，則訊息可能會以不同的語言顯示，具體取決於它們的來源。

**注意**\
當我們談到從執行環境繼承語言環境時，這意味著在大多數作業系統上都有以下內容：對於給定的語言環境類別，比如排序規則，將按此順序查詢以下環境變數，直到找到一個設定：LC\_ALL， LC\_COLLATE（或對應於相應類別的變數），LANG。如果未設定這些環境變數，則語言環境預設為 C.

某些訊息的本地化函式庫還會查看環境變數 LANGUAGE，該變數將覆寫所有其他區域設定，以便設定訊息的語言。如有疑問，請參閱作業系統的文件，特別是有關 gettext 的文件。

要使訊息能夠轉換為用戶的偏好語言，必須在編譯時選擇 NLS（`configure --enable-nls`）。所有其他語言環境支援都是自動編譯的。

## 24.1.2. 操作行為

語系設定會影響以下的 SQL 功能：

* 使用 ORDER BY 或標準比較運算子對查詢中文字排序
* upper，lower 和 initcap 功能
* 樣式匹配運算子（LIKE，SIMILAR TO 和 POSIX 形式的正規表示式）；locales 透過字元類的正規表示式影響不區分大小寫的匹配和字元分類
* to\_char 系列函數
* 索引可以與 LIKE 子句一起使用

在 PostgreSQL 中使用 C 或 POSIX 以外語言環境的缺點是對效能的影響。它會減慢字元處理速度並阻止 LIKE 使用普通索引。因此，最好只有在實際需要時才進行區域設定。

作為允許 PostgreSQL 在非 C 語言環境下使用具有 LIKE 子句索引的解決方法，存在多個自訂運算子類。允許建立一個執行嚴格的逐字元比較的索引，忽略區域設定的比較規則。有關更多訊息，請參閱[第 11.9 節](../../the-sql-language/index/operator-classes-and-operator-families.md)。另一種方法是使用 C collation 建立索引，如[第 24.2節](collation-support.md)中所述。

## 24.1.3. Selecting Locales

Locales can be selected in different scopes depending on requirements. The above overview showed how locales are specified using `initdb` to set the defaults for the entire cluster. The following list shows where locales can be selected. Each item provides the defaults for the subsequent items, and each lower item allows overriding the defaults on a finer granularity.

1. As explained above, the environment of the operating system provides the defaults for the locales of a newly initialized database cluster. In many cases, this is enough: If the operating system is configured for the desired language/territory, then PostgreSQL will by default also behave according to that locale.
2. As shown above, command-line options for `initdb` specify the locale settings for a newly initialized database cluster. Use this if the operating system does not have the locale configuration you want for your database system.
3. A locale can be selected separately for each database. The SQL command `CREATE DATABASE` and its command-line equivalent `createdb` have options for that. Use this for example if a database cluster houses databases for multiple tenants with different requirements.
4. Locale settings can be made for individual table columns. This uses an SQL object called _collation_ and is explained in [Section 24.2](https://www.postgresql.org/docs/15/collation.html). Use this for example to sort data in different languages or customize the sort order of a particular table.
5. Finally, locales can be selected for an individual query. Again, this uses SQL collation objects. This could be used to change the sort order based on run-time choices or for ad-hoc experimentation.

## 24.1.4. Locale Providers

PostgreSQL supports multiple _locale providers_. This specifies which library supplies the locale data. One standard provider name is `libc`, which uses the locales provided by the operating system C library. These are the locales used by most tools provided by the operating system. Another provider is `icu`, which uses the external ICU library. ICU locales can only be used if support for ICU was configured when PostgreSQL was built.

The commands and tools that select the locale settings, as described above, each have an option to select the locale provider. The examples shown earlier all use the `libc` provider, which is the default. Here is an example to initialize a database cluster using the ICU provider:

```
initdb --locale-provider=icu --icu-locale=en
```

See the description of the respective commands and programs for details. Note that you can mix locale providers at different granularities, for example use `libc` by default for the cluster but have one database that uses the `icu` provider, and then have collation objects using either provider within those databases.

Which locale provider to use depends on individual requirements. For most basic uses, either provider will give adequate results. For the libc provider, it depends on what the operating system offers; some operating systems are better than others. For advanced uses, ICU offers more locale variants and customization options.

## 24.1.5. 問題

如果區域設定依上述說明操作卻不起作用的話，請檢查作業系統中的區域設定是否已正確配置。要檢查作業系統上安裝的語言環境，可以使用命令 locale -a（如果作業系統有提供的話）。

檢查 PostgreSQL 實際上是否正在使用您認為的語言環境。LC\_COLLATE 和 LC\_CTYPE 設定會在建立資料庫時確定，除非建立新的資料庫，否則無法變更。其他區域設定（包括 LC\_MESSAGES 和 LC\_MONETARY）最初由伺服器啟動的環境決定，但可以即時變更。您可以使用 SHOW 命令檢查當下有效的區域設定。

原始碼發行版中的目錄 src/test/locale 包含了 PostgreSQL 語言環境支援的測試套件。

當伺服器的訊息使用不同的語言時，透過解析錯誤訊息文字來處理伺服器端錯誤的用戶端應用程序顯然會出現問題。建議此類應用程序的作者使用錯誤代碼方案。

維護訊息翻譯目錄需要許多志願者的持續努力，他們希望看到 PostgreSQL 能夠順暢地說出他們喜歡的語言。如果您的語言訊息目前無法使用或未完全翻譯，我們將非常歡迎您的協助。如果您想幫助我們，請參閱[第 54 章](../../internals/postgresql-coding-conventions/)或寫信給開發人員的郵件列表。

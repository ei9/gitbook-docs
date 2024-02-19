# B.4. 日期時間設定檔

由於時區縮寫並沒有很好地標準化，PostgreSQL 提供了一種自訂的伺服器接受的縮寫集方法。[timezone\_abbreviations](../../server-administration/server-configuration/client-connection-defaults.md#timezone\_abbreviations-string) 運行時參數決定有效的縮寫集。雖然此參數可由任何資料庫使用者變更，但其可能的值受資料庫管理員控制 - 實際上它們是儲存在安裝目錄的 .../share/timezonesets/ 中的組態檔案的名稱。透過增加或變更該目錄中的檔案，管理員可以為時區縮寫設定本地策略。

timezone\_abbreviations 可以設定為在 .../share/timezonesets/ 中找到的任何檔案名稱，檔案的名稱需要完全是英文字母的。（禁止 timezone\_abbreviations 中的非英文字母字元可以防止讀取目標目錄之外的檔案，以及讀取編輯器備份檔案和其他無關檔案。）

時區縮寫檔案可以包含空白行和以 # 開頭的註釋。非註釋行必須具有以下格式之一：

```
zone_abbreviation offset
zone_abbreviation offset D
zone_abbreviation time_zone_name
@INCLUDE file_name
@OVERRIDE
```

zone\_abbreviation 只是定義的縮寫。offset 是一個整數，定義與 UTC 相等的偏移量，以秒為單位，正向格林威治為東，負向為西。例如，-18000 將在格林威治以西 5 小時或北美東海岸標準時間。D 的區域名稱表示本地夏令時間而不是標準時間。

或者，可以定義 time\_zone\_name，引用 IANA 時區資料庫中定義的區域名稱。查詢區域的定義以查看該區域中是否正在使用縮寫，如果是，則使用適當的含義 - 即，目前正在確定其值的時間戳記中使用的含義，或者如果當時不是目前使用的話，則使用之前的含義，如果僅在該時間之後使用，則使用最舊的含義。這種行為對於處理其含義歷史變化的縮寫至關重要。還允許根據區域名稱定義縮寫，其中不出現該縮寫；使用縮寫就等於寫出區域名稱。

{% hint style="info" %}
在定義與 UTC 的偏移量從未改變的縮寫時，偏好使用簡單的整數偏移量，因為這些縮寫比需要查閱時區定義的縮寫要簡單得多。
{% endhint %}

@INCLUDE 語法允許在 .../share/timezonesets/ 目錄中包含另一個檔案。包含可以嵌套到某個有限的深度。

@OVERRIDE 語法指示檔案中的後續項目可以覆蓋先前的項目（通常是從包含的檔案中獲取的項目）。如果沒有這個，相同時區縮寫的衝突定義將被視為錯誤。

在未修改的安裝環境中，檔案 Default 包含了世界上大多數地區的所有非衝突時區縮寫。為這些區域提供了澳大利亞和印度的附加檔案：這些檔案先有預設設定，然後根據需要增加或修改縮寫。

出於參考目的，標準安裝還包含 Africa.txt，America.txt 等文件，其中包含有關根據 IANA 時區資料庫已知正在使用的每個時區縮寫的訊息。可以根據需要將這些檔案中找到的區域名稱定義複製並貼到自行定義配置檔案中。請注意，由於名稱中包含了點，因此無法將這些檔案直接引用為 timezone\_abbreviations 設定。

{% hint style="info" %}
如果在讀取時區縮寫集時發生錯誤，則不會應用新值並保留舊值。如果在啟動資料庫時發生錯誤，則啟動失敗。
{% endhint %}

{% hint style="info" %}
配置檔案中定義的時區縮寫會覆寫 PostgreSQL 中內建的非時區定義。例如，澳大利亞配置檔案定義了 SAT（南澳大利亞標準時間）。當此檔案有效時，SAT 將不會被識別為星期六的縮寫。
{% endhint %}

{% hint style="info" %}
如果您修改 .../share/timezonesets/ 中的檔案，則由您來進行備份 - 正常的資料庫轉存將不包含此目錄。
{% endhint %}


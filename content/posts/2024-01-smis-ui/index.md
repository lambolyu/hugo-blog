+++
title = '智慧防疫物資管理系統：介面'
slug = '2024-01-smis-ui'
date = 2024-01-04T13:07:58+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['PHP','Javascript']
tags = []
+++
上個月分別建立了關於智慧防疫物資管理系統的資料比對與通報檔案生成的功能，剩下的就是製作一個可以讓使用者互動的介面了！

這個介面為了讓各台電腦不用安裝執行檔，決定採用 web 的方式呈現，所以就使用 php 做為後端資料庫溝通、後端 csv 檔案讀取，加上一點少少的 JavaScript 強化使用者體驗。
***
## 網頁切版
為了求製作快速，我使用了慣用的 [Bootstrap](https://getbootstrap.com/) 簡單用裡面的 grid 功能進行網頁切版，規劃的概念圖如下：
{{< figure src="div-design.png" width="65%" alt="page-block-design" align="center" >}}

html 代碼如下：
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="" crossorigin="">
    <title>SMIS 通報介面</title>
  </head>
  <body>
    <div class="container">
      <div class="row"> <!--標題--->
        <div class="col">
        </div>
      </div>
      <div class="row"> <!--庫存和趨勢--->
        <div class="col">
        </div>
      </div>      
      <div class="row"> <!--待通報區域--->
        <div class="col">
        </div>
      </div>
      <div class="row"> <!--已通報區域--->
        <div class="col">
        </div>
      </div>
    </div>
  </body>
</html>
```

>之前為了快速建構網頁的面板，我使用 Bootstrap 都是以 cdn 的方式開始，不過這幾個月因為院方希望把我的東西轉到內網，呼叫 cdn 必然受到限制，因此我最近開始考慮學習比較現代化的前端開發 Orz 。

***
## 確認 csv 更新了沒
前面的文章有提到，我利用 python 匯出院內報表後形成一個「三至四個月的所有資料」並且命名成 `smis.csv` ，以便後續 php 的比對。後續上線後，曾經因為院內程式更改 UI 導致 python 自動匯出失敗，然而這個錯誤並沒有任何的提示，使用者因此漏報了兩三天才發現問題。

因此必須在頁面上顯示 `smis.csv` 更新時間，以防止院內程式的自動化失效。

檔案的時間標記分成三種：
- atime(**A**ccess time)：檔案上次被讀取的時間。
- ctime(status **C**hange time)：檔案的屬性或內容上次被修改的時間。
- mtime(**M**odified time)：檔案的內容上次被修改的時間。

以這邊的例子來說， python 程式自動形成檔案後，這三個時間都會一樣，但是 atime 可能會受到其他程式讀取而有所改變，因此要顯示更新時間最好使用 mtime 或 ctime ，可以利用 `filemtime` 方法把檔案的修改時間叫出來，但因為返回的時間值是 Unix 時間戳的格式，因此必須再用 `date` 將時間戳轉換成可閱讀的格式：
```php
<?php
$csvmtime = date("n/j H:m", filemtime("smis.csv"));
```
***
## 院內剩餘庫存
除了前面文章提到計算「待通報病人」跟將「已通報病人」的檔案下載後批次上傳 CDC 網頁之外，使用者也需要知道目前院內的庫存以核對 CDC 的網頁，因此必須建立一個額外的資料庫來存放藥物的剩餘庫存數量：
```mysql
CREATE TABLE `smis_stock` (
  `id` int(11) NOT NULL,
  `medicine` varchar(20) NOT NULL,      /*藥品名稱*/
  `quantity` mediumint(9) NOT NULL,     /*異動數量*/
  `action` varchar(10) NOT NULL,        /*異動原因*/
  `happendate` int(11) NOT NULL,        /*異動日期*/
  `insertdate` int(11) NOT NULL         /*資料新增日期*/
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

庫存數量會利用「異動原因」來做分類計算，「異動原因」共分成三種情形：
- 結餘：宣告所有數量由此筆資料的日期以後進行計算。
- 入庫：政府撥補藥品給醫院。
- 退庫：政府回收藥品。

舉個例子：
```csv
id,medicine,quantity,action,happendate,insertdate
...
30,Paxlovid,20,入庫,20231001,...
31,Paxlovid,1520,結餘,20231003,...
32,Paxlovid,200,入庫,20231004,...
33,Paxlovid,-500,退庫,20231005,...
34,Paxlovid,40,入庫,20231010,...
35,Paxlovid,-60,退庫,20231025,...
```

Paxlovid 在 2023 年 10 月 03 日的結餘量是 1520 盒，「結餘」這個原因宣告所有數量由此筆資料的日期以後進行計算，也就是說，第 30 筆資料不計，目前的庫存應該是 `1520 + 200 + (-500) + 40 + (-60)` ，結餘量加上所有的進出量。

```php
//目前庫存量
//找到各自的最後一個結餘日期
$fetchrows = $conn->query("SELECT `medicine`, `happendate` FROM `smis_stock` WHERE `action` = '結餘'")->fetchAll(PDO::FETCH_ASSOC);
$checkdate = array_column($fetchrows, "happendate", "medicine");
//庫存總和
foreach ($checkdate as $medicine => $happendate) {
    $fetchrows = $conn->query("SELECT SUM(`quantity`) FROM `smis_stock` WHERE `medicine` = '$medicine' AND `happendate` >= '$happendate'")->fetch(PDO::FETCH_NUM);
    $stock[$medicine] = $fetchrows[0];
}
```

計算出來的數字是所謂的`初始庫存`，而`已通報資料庫`中也可以加總計算「結餘日期」之後的使用數量，相減之後就是目前的剩餘庫存了。
```php
//扣除耗用
foreach ($checkdate as $medicine => $happendate) {
    $fetchrows = $conn->query("SELECT SUM(`quantity`) FROM `smis` WHERE `medicine` = '$medicine' AND `reportdate` >= '$happendate' AND `notreport` = '無' ")->fetch(PDO::FETCH_NUM);
    $inventory[$medicine] = $stock[$medicine] - $fetchrows[0];
}
```
***
## 待通報區域
![waiting to report div](prereport.png#center)
***
## 手動通報
***
## 已通報區域
***
## ~~使用趨勢圖表~~
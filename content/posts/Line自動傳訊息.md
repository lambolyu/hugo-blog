+++
title = 'Line 自動傳訊息到群組'
slug = 'Line-post-to-group'
date = 2023-10-02T08:42:10+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['工具小零件']
tags = []
+++
醫院藥庫有許多資訊需要被公布，例如說藥品的包裝有重大的變更，或者是藥品因為缺貨，需要暫時以其他藥品替代等等。因此藥庫在醫院內網建置了一個網頁布告欄，隨時將最新消息和各式附件張貼在網頁布告欄上。

然而並不是所有的藥師都有時間到院內電腦連上網頁布告欄觀看訊息，為了能夠即時將重要的資訊轉達給所有藥師，因此使用 Line 這個臺灣絕大多數人都會使用的即時通訊軟體進行訊息的傳送，成為了比較好的解決辦法。

***
## 程式流程
原本使用網頁布告欄的流程如下：
<!--![Stock Bulletin Flow](/images/2023-10-stock-notify-original.png#center)-->
{{< figure src="/images/2023-10-stock-notify-original.png" width="60%" alt="Stock Bulletin Original Flow" align="center" >}}
藥庫藥師將資訊以 PHP 網頁對資料庫新增一筆紀錄，線上藥師再以 PHP 網頁觀看資料庫的紀錄，此處的 PHP 網頁的頁首有自動查詢前幾筆的功能。

若要使用 Line 自動發送訊息，規劃流程如下：
{{< figure src="/images/2023-10-stock-notify-line.png" width="90%" alt="Stock Bulletin Line Flow" align="center" >}}
藥庫藥師將資訊以 PHP 網頁對資料庫新增一筆紀錄，此時 PHP 會自動篩選紀錄，如果紀錄比較重要，會複製一份起來傳送到某個雲端。
該雲端會使用 Line 發送訊息的形式傳送到線上藥師所在的 Line 群組中。

同時紀錄也留在原本的資料庫中，所以線上藥師亦可以使用 PHP 網頁觀看資料庫的紀錄。
{{< figure src="/images/2023-10-stock-notify-mix.png" width="90%" alt="Stock Bulletin Full Flow" align="center" >}}
為了避免資料庫中的訊息外流，雲端不直接讀取資料庫，只在藥庫藥師新增紀錄時觸發程式。優點是可以減少資安的疑慮，缺點是除非再次新增紀錄，否則無法重新發送資訊。
***
## Line Bot
![Line Messaging API](https://developers.line.biz/zh-hant/_media/services/messaging-api-thumb0.png)

說到使用 Line 自動發送訊息，最先想到的就是利用 Line Bot ，這個被各大企業公司用來當成客服的聊天機器人。Line Bot 準確來說是 Line Messaging API ，可以主動對與機器人成為好友的使用者推送訊息，也能夠以程式的方式回應使用者對機器人說的話。

後來實作了一陣子發現了一些優點但也發現了缺點，列舉如下：
### 優點
- 可以自訂機器人的大頭貼圖片和名稱
- 具有後台可以不用寫程式就可以自動回應加入好友通知和自動以 AI 回應字句
- 具有後台可以觀看被加好友、回應次數等視覺化的圖表
- 除了 Messaging API ，還有更多例如商家集點、連結 Line Pay 服務等功能
- 說明文件清楚，並開發有 SDK 可供開發使用，而且開發者眾多網路資源豐富
### 缺點
- 需要找一個雲端平台部署，並讓 Line Messaging API 連結該平台
- 免費額度為每個月 200 則訊息免費，同一則訊息發到 5 個群組內算 5 則

>2023/10/03 Line Messaging API 共分三種資費：輕用量免費 Quota 200則/月，超過無法使用、中用量 800 元/月 Quota 3000則/月，超過無法使用、高用量 1200元/月 Quota 6000則/月，超過每則 0.2 元。

當初藥庫的布告機器人是利用 Line Messaging API 建置後部署在 [Heroku](https://www.heroku.com/) 雲端平台上運行，但其實面臨到一些問題。

{{< figure src="https://cdn-icons-png.flaticon.com/512/873/873120.png" width="10%" alt="Heroku" align="center" >}}

Heroku **當時**的免費方案是超過 30 分鐘沒有使用會進入休眠狀態，再次觸發要等 30 秒左右的甦醒時間，有些時候可能會漏掉休眠時送過去的資料。後來便以 [kaffeine](https://kaffeine.herokuapp.com/) 等服務每 30 分鐘就戳一下機器人，讓 Heroku 不要睡著，看起來改善很多。

後來隨著 Heroku 開始收費，加上本院藥學部的群組變多了，每個月接收到的訊息也變多了，幾天之內就會超過 Line Messaging API 的免費額度，由於布告機器人並不是醫院的預算案，~~而且也不希望扯上醫院的預算案~~，於是斷然放棄了機器人。

>2022/11/28 Heroku 宣布取消免費方案，罵聲不斷後雖然一度又宣布取消，但目前 看來最低還是有 5 美元/月的收費。

不過雖然機器人不能作為發送布告訊息的用途，還是有其他方法可以使用，這個會是另一篇文章的主題。

***
## Line Notify
![Line Notify](/images/2023-10-linenotify.png#center)
***
## Attribution
- [Icons created by Freepik - Flaticon](https://www.flaticon.com/)

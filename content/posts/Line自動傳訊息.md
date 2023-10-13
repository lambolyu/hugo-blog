+++
title = 'Line 自動傳訊息到群組'
slug = 'Line-post-to-group'
date = 2023-11-02T08:42:10+08:00
draft = false
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['Line','PHP']
tags = ['Line','Line Bot','Line Notify','PHP','Python','自動傳訊息','推播']
[cover]
image = '/images/2023-10-linenotify-cover.png'
+++
醫院藥庫有許多資訊需要公布，例如說藥品的包裝有重大的變更，或是因為缺貨，需要暫時以其他藥品替代等等。因此藥庫在醫院內網建置了一個網頁布告欄，隨時將最新消息和各式附件張貼在網頁布告欄讓其他藥師閱覽。

然而並不是所有的藥師都有時間到院內電腦連上網頁布告欄觀看訊息，為了將重要的資訊轉達給所有藥師，因此使用 Line 這個臺灣大多數人都在使用的通訊軟體進行訊息的傳送，是一個比較即時又有效率的辦法。

***
## 程式流程
原本網頁布告欄的流程如下：
<!--![Stock Bulletin Flow](/images/2023-10-stock-notify-original.png#center)-->
{{< figure src="/images/2023-10-stock-notify-original.png" width="60%" alt="Stock Bulletin Original Flow" align="center" >}}
藥庫藥師將資訊以 PHP 網頁發文到資料庫中，線上藥師再以 PHP 網頁觀看文章。

若要使用 Line 自動發送訊息，規劃如下：
{{< figure src="/images/2023-10-stock-notify-line.png" width="90%" alt="Stock Bulletin Line Flow" align="center" >}}
藥庫藥師將資訊以 PHP 網頁發文到資料庫中，經過 PHP 篩選重要的內容，複製並傳送到某個雲端。
該雲端再以 Line 的形式傳送到線上藥師所在的 Line 群組中。

同時文章也留在原本的資料庫中，所以線上藥師亦可以使用 PHP 網頁文章。
{{< figure src="/images/2023-10-stock-notify-mix.png" width="90%" alt="Stock Bulletin Full Flow" align="center" >}}
為了避免資料庫中的資訊外流，雲端不直接讀取資料庫，只在藥庫藥師發文時觸發程式。優點是可以減少資安的疑慮，缺點是除非再次發文，否則無法重新傳 Line。
***
## Line Bot
![Line Messaging API](https://developers.line.biz/zh-hant/_media/services/messaging-api-thumb0.png)

說到使用 Line 自動發送訊息，最先想到的就是利用 Line Bot ，這個被各大企業公司用來當成客服的聊天機器人。Line Bot 準確來說是 Line Messaging API ，可以主動對與機器人成為好友的使用者推送訊息 (Push messages) ，也能夠以自動回應使用者 (Reply messages) 。

後來實作了一陣子發現了一些優點但也發現了缺點，列舉如下：
### 優點
- 可以自訂機器人的顯示圖片和名稱
- 後台可以直接設定自動回應預設字句或以 AI 分析使用者留言後回應
- 具有後台可以觀看被加好友、回應次數等視覺化的圖表
- 除了 Messaging API ，還有更多例如商家集點、連結 Line Pay 服務等功能
- 說明文件清楚，還有官方提供的 SDK 可供開發使用，而且開發者眾多，網路資源豐富
### 缺點
- 需要找一個雲端平台部署，不熟悉的新手可能會小卡關
- 免費額度為每個月 200 則訊息免費，同一則訊息發到 5 個群組內算 5 則

>2023/10/03 Line Messaging API 共分三種資費：輕用量免費 Quota 200則/月，超過無法使用、中用量 800 元/月 Quota 3000則/月，超過無法使用、高用量 1200元/月 Quota 6000則/月，超過每則 0.2 元。

當初藥庫的通知機器人是利用 Line Messaging API 建置後部署在 [Heroku](https://www.heroku.com/) 雲端平台上運行，但其實面臨到一些問題。

{{< figure src="https://cdn-icons-png.flaticon.com/512/873/873120.png" width="10%" alt="Heroku" align="center" >}}

Heroku **當時**的免費方案是超過 30 分鐘沒有使用會進入休眠狀態，再次觸發要等 30 秒左右的甦醒時間，有些時候可能會漏掉休眠時送過去的資料。後來便以 [kaffeine](https://kaffeine.herokuapp.com/) 等服務每 30 分鐘就戳一下機器人，讓 Heroku 不要睡著，體驗上改善很多。

隨著 Heroku 開始收費，加上本院藥學部的群組變多了，每個月需要公告通知的訊息也變多了，幾天之內就會超過免費額度，由於通知機器人並不是醫院的預算案，~~而且也不希望扯上醫院的預算案~~，於是斷然放棄了 Line Bot 。

>2022/11/28 Heroku 宣布取消免費方案，罵聲不斷後雖然一度又宣布恢復免費，但目前看來最低還是有 5 美元/月的收費。

雖然 Line Bot 對於現在的藥庫已經不能作為通知布告的用途，還是有其他地方可以派上用場，這個會是另一篇文章的主題。

***
## Line Notify
![Line Notify](/images/2023-10-linenotify.png#center)
替代方案就是 Line 提供的另外一項免費的服務： Line Notify ，說明文件請參考[這裡](https://notify-bot.line.me/doc/en/)。

有別於 Line Bot ， Line Notify 本身沒有後台可以觀看數據，無法更改顯示圖片和名稱，名稱會預設以【○○○】夾註並在後面顯示推送的訊息。

![Line Notify Preview](/images/2023-10-linenotify-preview.png#center)

而且 Line Notify 只能發送訊息，無法得知對方回覆了什麼，也無法知道群組裡的對話。但是對於通知布告的用途卻已經足夠了。

### 1. 登入 Line 帳號
![login line account](/images/2023-10-linenotify-0.png#center)
點選右上角登入自己個人的 Line 帳號，這個帳號是個人名義或其他名義登錄的都沒有差別，不會在任何地方顯示出來。接下來的文章會把這個登入的帳號叫做 「設定人」 。

### 2. 管理登錄服務
![管理登錄服務](/images/2023-10-linenotify-1.png#center)
選擇管理登錄服務。

這個選項會有一點小複雜，以下的步驟以 PHP 為例，其他還有搭配 ngrok 的例子，可以參考 The Will Will Web 保哥的[文章](https://blog.miniasp.com/post/2020/02/17/Go-Through-LINE-Notify-Without-Any-Code)。

首先要找到或是建立網頁伺服器，無法連接外網也無所謂，如果使用 PHP ，最簡單的方法應該是使用 [XAMPP](https://www.apachefriends.org/) 來快速建立伺服器環境，相關的教學內容可以自行 Google ，伺服器的意思就是最後在瀏覽器上輸入 `http://localhost/` 或是 `http://127.0.0.1/` 或是指定的 IP 位置可以指向某個資料夾裡的 PHP 網頁。

藥庫本身的布告欄是架在院內的某台伺服器上，因此可以直接挪用伺服器空間。

#### A. 填寫資料完成取得 Client ID 和 Client Secret
![管理登錄服務](/images/2023-10-linenotify-2.png#center)
選擇管理登錄服務後，進入到填寫資料的畫面，這邊所有的空格都是必填的。比較重要的項目其實只有三個，其他的都不會公開。
- 服務名稱：【○○○】內的文字
- 電子郵件帳號：該步驟完成後 Line 會寄認證信要求完成認證
- Callback URL：這個可以先亂填一個 http 開頭的網址

寫錯或是亂寫都沒有關係，這些資訊之後都可以隨時更改。寫完之後前往下一步。

![管理登錄服務](/images/2023-10-linenotify-4.png#center)
再次確認資訊，按下登錄。

![管理登錄服務](/images/2023-10-linenotify-5.png#center)
回到自己的信箱點選認證信，再點選前往服務一覽。

![管理登錄服務](/images/2023-10-linenotify-6.png#center)
點選剛剛建立的服務，可以取得 Client ID 和 Client Secret 。

![管理登錄服務](/images/2023-10-linenotify-6-1.png#center)
　
#### B. 製作 Call Back 頁面
接著來製作讓 Line Notify 認證回傳的頁面。

根據說明文件，中間藍色的 YOUR SITE 就是我們要製作的頁面。
![linenotify oauth flow](https://scdn.line-apps.com/n/line_notice/img/pc/img_api_document1.png#center)

當我們向認證的 API 發出請求並選擇特定群組的時候 (圖片上方藍色箭頭) ，該 API 會重新導向我們指定的 CallBack URL ，並用 POST 方法發送參數給 CallBack URL  (圖片上方綠色箭頭) 。

而我們必須使用剛剛拿到的參數，再次 POST 給另外一個 Line Notify API (圖片下方藍色箭頭) ，以此拿到特定群組的 Access Token (圖片下方綠色箭頭) 。

很複雜看不懂沒關係，網頁的原始碼如下，抄起來貼上，上面的 `const` 放上剛剛查詢到的 Client ID 和 Client Secret ，然後把檔案名稱存成 callback.php 放在伺服器根目錄中：
```php
<?php
// client id
const CLIENT_ID = '';
// client secret
const CLIENT_SECRET = '';
// callback URL，連到這一頁的網址名稱
const AUTH_PAGE_URL = 'http://localhost/callback.php';

if(isset($_POST['code'])) {
    $ch = curl_init();
    curl_setopt_array($ch, [
        CURLOPT_URL => 'https://notify-bot.line.me/oauth/token',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => [
            'grant_type' => 'authorization_code',
            'code' => $_POST['code'],
            'redirect_uri' => AUTH_PAGE_URL,
            'client_id' => CLIENT_ID,
            'client_secret' => CLIENT_SECRET
        ],
        CURLOPT_SSL_VERIFYHOST => 0,
        CURLOPT_SSL_VERIFYPEER => 0,
    ]);
    $body = curl_exec($ch);
    $code = curl_getinfo($ch,  CURLINFO_RESPONSE_CODE);
    curl_close($ch);

    if($code===200) {
        $json=json_decode($body, true);
        echo 'access_token: '.(isset($json['access_token']) ? $json['access_token'] : 'null');
    } else {
        echo '無法取得 access_token';
    }

} elseif(isset($_POST['error']) && isset($_POST['error_description'])) {
    exit($_POST['error'].'<br>'.$_POST['error_description']);
} else {
    exit('未知的錯誤');
}
```
#### C. 修改剛剛資料頁中的 Callback URL
這邊要跟剛剛頁面上定義的常數值要一模一樣，要檢查 http 後面有沒有 s ，網誌後面有沒有 / 等等。
![管理登錄服務](/images/2023-10-linenotify-6-2.png#center)
#### D. 完成 OAuth2 認證取得群組的 Access Token
複製下面的網址，並更改網址中的資訊。
```
https://notify-bot.line.me/oauth/authorize?response_type=code&scope=notify&state=0&client_id=&redirect_uri=
```
- 在 `client_id=` 和 `&` 之間放上自己申請的 client id
- 在 `redirect_uri=` 之後放上 `http://localhost/callback.php` 或自己的 Callback URL

輸入網址後看到下面這個畫面，表示成功了！在選單上選擇想讓 Line Notify 傳送訊息 (連動) 到哪個群組，這裡的選項只有設定人自己跟設定人存在的群組而已，不過取得 Access Token 之後，設定人可以退出群組， Line Notify 依舊會有作用。
![管理登錄服務](/images/2023-10-linenotify-7.png#center)

按下按鈕後，瀏覽器就會回到 `http://localhost/callback.php` ，並在該頁面上顯示該群組的 `access_token` ，請小心保存下來，這個 Token 只會顯示這一次，如果不小心遺失了就必須重新發行一次。

### 2. 個人頁面
如果上面的流程太複雜了，那來試試這個。

剛剛登入 Line 之後，這次改選個人頁面。
![開發人員用存取權杖](/images/2023-10-linenotify-8.png#center)

網頁最下面選擇發行開發人員用的存取權杖。
![開發人員用存取權杖](/images/2023-10-linenotify-9.png#center)
#### A. 填寫資料完成取得 Access Token
接著填入自訂的權杖名稱【○○○】，選擇想讓 Line Notify 傳送訊息 (連動) 到哪個群組，點選發行。
![開發人員用存取權杖](/images/2023-10-linenotify-10.png#center)

這樣 Access Token 就出現了。是不是有夠簡單？
![開發人員用存取權杖](/images/2023-10-linenotify-11.png#center)

#### B. 管理登錄服務 vs 個人頁面
那為什麼要選擇**管理登錄服務**，捨近求遠往艱難的路走呢？

差別在於後續能不能改名稱，也就是改【○○○】。如果使用個人頁面發行存取權杖，權杖名稱【○○○】當下就必須決定好，日後無法更改，如果需要更改就必須重新發行一次。而管理登錄服務，則可以隨時變更名稱，甚至可以利用同一個登錄服務發行多個群組的權杖，統一管理也比較有一致性。

講了很多優點，我個人還是覺得使用個人頁面直接發行單一次權杖好用很多。不過因為醫院的有些群組涉及機密或考慮管理層面，部分主管不喜歡額外的人員 (就是我) 留在他們的群組中，但又必須將藥庫通知機器人留著，所以變成我設定好之後就退出群組，如此一來就沒有重新發行權杖的機會，因此對我而言，使用管理登錄服務會是比較好的選擇。
![已退出的群組](/images/2023-10-linenotify-kickgroup.png#center)

### 3. 將 Line Notify 加入群組
發行權杖或是取得 Access Token 之後，設定人的 Line 就會收到 Line Notity 的通知，把這個 Line Notify 拉進剛剛設定的群組裡。
![新增 Linenotify 進群組](/images/2023-10-linenotify-12.png#center)

### 4. 利用 curl 傳送訊息
現在只需要利用 POST 方法發送訊息就可以觸發 Line Notify 跟群組之間的關係了。
#### 以 python 為例
使用 python requests 函式庫就可以直接操作：
```python
import requests
access_token = "0PNUiBQqXyHV******TlJxpjNe5Kv5hoIJty7O"
# Request headers
headers = { 
    "Content-Type": "multipart/form-data",
    "Authorization": "Bearer " + token 
    }
# Request parameters
# 要發送的訊息，前面有一個換行的符號，因為我覺得【○○○】後面應該要換一行再傳訊息比較有辨識度。
data = {
    "message": "\n測試測試"
    }
# 以 requests 發送 POST 請求
requests.post("https://notify-api.line.me/api/notify", headers = headers, data = data)
```
#### 以 PHP 為例
而 PHP 要使用一套 curl 相關的方法來操作：
```php
<?php
$access_token = "0PNUiBQqXyHV******TlJxpjNe5Kv5hoIJty7O";
# Request headers
$headers = [
        "Content-Type: multipart/form-data",
        "Authorization: Bearer ".$access_token
    ];
# Request parameters
$data = ["message" => "
"."測試測試"];
# 發送 POST 請求
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "https://notify-api.line.me/api/notify");
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
curl_exec($ch);
curl_close($ch);
```
### 4. 連結布告欄系統
於是在原本布告欄系統新增文章的行為完成後，利用變數決定要不要接著觸發機器人，並且因為布告欄系統存在的伺服器無法連結外網，發送 POST 的網頁必須找一台可以連接外網的伺服器，因此寫成這樣：
```php
<?php
// 布告欄系統

// 新增文章
...
$stmt = $conn->...;
$stmt->execute(...);
$insert_id = $conn->lastInsertId();

if ($line!="") {
    //群組設定
    $group = "opd-ud";
    //訊息設定 藥名：資訊
    $msg = $medicine_name."：".$content;
    //過濾非法字元
    $msg = str_replace("#", "", $msg);
    //連線到另外一台可以連外網的伺服器
    header("location: http://可以連外網的伺服器/linenotify.php?id=".$insert_id."&group=".$group."&msg=".$msg);
}
```
>Line Notify 發出訊息時 # 字號和後面的文字都不會出現，所以訊息中不能有 # 字號。

接著是放在可以連外網伺服器的 `linenotify.php` ：
```php
<?php
$group_list = explode("-", $_GET["group"]);
$group_token = [
    "opd" => "URn5NviA5g52T1*****usDEelgS3TNeb8f5nuo3eKRc", //門診
    "ud" => "sLZVqcGPuVPpQQ4w6NMX*****9aJpFsqJxPmNlPZUeV", //住院
    "adm" => "5LIZ1Vyxp*****TnkqpojFPvybyxi2il3TAC9xMHmw0", //行政
];
$message = ["message" => "
".$_GET["msg"]];

foreach ($group_list as $group) {
    $headers = [
        "Content-Type: multipart/form-data",
        "Authorization: Bearer ".$group_token[$group]
    ];
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, "https://notify-api.line.me/api/notify");
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $message);
    curl_exec($ch);
    curl_close($ch);
}
// 推播完訊息返回布告欄頁面
header("Location: http://布告欄所在的伺服器/bulletin.php?edit=".$_GET["id"]); 
```
大功告成了！
***
## Attribution
- [Icons created by Freepik - Flaticon](https://www.flaticon.com/)
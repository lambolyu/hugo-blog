+++
title = '藥品數量與包裝的換算'
slug = '2023-09-medicine-package-quantity'
date = 2023-09-20T10:05:30+08:00
draft = false
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['Python','PHP']
tags = ['Python','MySQL','Database','Javascripts','數量換算','包裝量','調藥系統','報表優化','藥品包裝']
+++
## 藥品的包裝
因為台灣醫療改革基金會推行，民眾領到的藥品將以「原包裝」為主。例如原廠包裝 1 盒為 28 顆裝的藥品，當醫師開立超過 28 顆時，藥師就必須發出 1 盒原廠包裝搭配手剪散藥給予民眾。

但是藥品百百種，各家廠商決定包裝的時候可能會參考**藥品的使用頻率**或**生產的利益效率**，決定以盒裝或瓶裝生產，盒裝又會因為藥品的使用方法等因素而有不同形式的鋁箔片，而瓶裝藥品則可能有 100 顆裝或 1000 顆裝等情形。

>臺灣或歐美等醫院，因為醫師看診是固定在星期幾，民眾回診時間也因此為 7 的倍數。例如民眾可能在禮拜三看診，下次回診如果沒有特別調整或是換醫師，應該還是禮拜三。因此大部分藥盒或是藥片數量設計都是 7 的倍數。
>
>又：因為這樣的緣故，醫院或處方上面的 1 個月，通常指的是 28 天而不是 30 天。

盒裝數量或瓶裝數量已經很多種了，但是對於藥庫而言，購買的包裝卻是以「箱」計算，而各家廠商每箱內所含的盒數或是瓶數更是達到百家爭鳴的盛況。

為了解決各個藥品具有不同包裝量的情形，我們決定將藥品各層包裝分級處理。例如膠囊或是錠劑的藥品，第一級包裝即為「顆」，包裝量為 1 ；第二級包裝可能為「片」，表示該藥品為一片鋁箔片，鋁箔片上會有一個囊泡一顆藥品的包裝設計，而鋁箔片上的囊泡數量就是包裝量，可能為 7 或 14 ，也有可能是 10 或 15 ；第三級包裝可能為「盒」，藥盒內裝載一個月份量的鋁箔片，如果第二級數量是 14 ，藥盒中可能就有 2 片，或可能因為藥品是一天吃兩次的，藥盒中就會有 4 片等等。
***
## 資料表設計
為了加速藥庫對於藥品數量的計算，我們決定以資料庫的方式紀錄藥品的包裝資訊，並且設計一個可以容納前面提過各級包裝單位和包裝量的 Schema (資料表及欄位的設計)。

以藥品 [Bokey 伯基腸溶微粒膠囊](https://info.fda.gov.tw/MLMS/H0001D.aspx?Type=Lic&LicId=01037344) 為例，每片鋁箔片有 28 顆膠囊，每盒內有 35 片鋁箔片，每箱內有 20 盒藥品。
<!--![Bokey](https://druginfo.cmuh.org.tw/drugpic/TASPI100.jpg)-->
{{< figure src="https://druginfo.cmuh.org.tw/drugpic/TASPI100.jpg" width="70%" alt="Bokey Images" align="center" >}}

資料表設計如下：

|自動序號|藥品代碼|藥品名稱|包裝層級|包裝名稱|包裝數量|建立時間|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|...|BOK001|Bokey 100mg Cap|1|顆|1|20230911131400|
|...|BOK001|Bokey 100mg Cap|2|片|28|20230911131528|
|...|BOK001|Bokey 100mg Cap|3|盒|35|20230911131730|
|...|BOK001|Bokey 100mg Cap|4|箱|20|20230911132001|

為了直覺的新增資料，包裝數量的定義是**該層級內含多少上一層級的數量**，以 Bokey 的例子來說，層級 3 的**盒**包裝數量 35 ，表示盒內具有 35 個層級 2 的包裝，也就是 35 **片**。

資料表的結構如下：
```MySQL
CREATE TABLE `pkgspec` (
  `id` int(11) NOT NULL,
  `code` varchar(6) NOT NULL,
  `name` varchar(50) NOT NULL,
  `speclevel` tinyint(2) NOT NULL,
  `specname` varchar(4) NOT NULL,
  `specqty` int(11) NOT NULL,
  `buildtime` varchar(14) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

ALTER TABLE `pkgspec`
  ADD PRIMARY KEY (`id`);

ALTER TABLE `pkgspec`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;
COMMIT;
```
***
## 資料表應用：小變大
大部份的使用情形是**將藥品顆數轉為包裝量**。例如事務員進行藥品調度時，院內系統產生的調度單上面會寫：`需調度 58,800 顆 Bokey 100mg Cap 至門診藥局`。於是事務員就必須先計算 58,800 顆是幾箱藥品，才能進行調度作業。

### Python
於是可以先將院內系統產生的調度單形成 csv 檔後，利用 Python Pandas 的 DataFrame 匯入檔案處理數字。

處理數字的部分，就是一連串的除法：
```python
import pandas as pd
import mysql.connector
from sqlalchemy import create_engine

code = 'BOK001'
qty = 58800

sql = f"SELECT * FROM pkgspec WHERE `code` = '{code}' ORDER BY `speclevel`,`buildtime` ASC"
engine = create_engine('mysql+mysqlconnector://使用者名稱:密碼@伺服器存在位置IP:埠號/資料庫名稱')
df = pd.read_sql(sql, con=engine)

df = df.sort_values(['speclevel','buildtime']).drop_duplicates('speclevel', keep='last')
#避免重複建立的包裝量，因此以speclevel為索引刪掉重複值，並且留下最新更新的資料

df['specqty'] = df['specqty'].cumprod() #累乘
```
核心方法就是 `cumprod()` ，預設參數是 `axis=0` 向下累乘，效果如下，第三行的 980 就是 1 * 28 * 35，而 19600 = 1 * 28 * 35 * 20：
![Pandas cumprod()](df-cumprod.png#center)

再將 DataFrame 轉換成列表 `[['箱', 19600], ['盒', 980], ['片', 28], ['顆', 1]]` ，操作起來比較直覺：
```python
df = df.sort_values('speclevel', ascending=False)[['specname','specqty']].values.tolist()
```
接下來就把 58,800 丟進迴圈裏面相除取餘數、餘數再除取餘數直到列表除完：
```python
output = '' #指定一個變數來接結果
if len(df) == 0:
    output = f'{qty}？'
elif qty <= 0:
    output = f'{qty}{df[-1][0]}'
else:
    first = True
    for specname,specqty in df:
        if first: 
            quotient = qty // specqty
            remainder = qty % specqty
            prevremainder = remainder
            first = False
            if quotient > 0:
                output += f'{quotient}{specname}'
        else:        
            quotient = prevremainder // specqty
            remainder = prevremainder % specqty
            prevremainder = remainder
            if quotient > 0:
                output += f'{quotient}{specname}'
```
程式完成後，用 `def` 和 `return` 包裝成自訂函式：
```python
def pkgcalc(code, qty):
    ...
    return output
```
然後回到原本讀入 csv 的報表程式中，利用 lambda 表達式帶入自訂函式：
```python
df = pd.read_csv('報表路徑')
df = df[['藥品代碼','藥品名稱','調度量']]
df['調度包裝'] = df.apply(lambda x: pkgcalc(x['藥品代碼'], x['調度量']), axis=1)
```
如此就可以在報表中直接顯示成 `需調度 5 箱 Bokey 100mg Cap 至門診藥局` ，一目了然。
### PHP
有時候會需要在網頁報表上直接呈現包裝量，因為目前所有的網頁報表都是使用 PHP 加上少部分的 javascript 完成的，如果今天使用 django 架構網站，便可以直接利用以上函式轉換。

所以必須將剛剛的 Python 用 PHP 重新寫一次：
```php
function pkgcalc($code, $qty){
    //連接資料庫
    $conn = new PDO("mysql:host=;dbname=;charset=", "帳號", "密碼");
    $conn->exec("SET NAMES utf8");
    $sql = "SELECT * FROM pkgspec WHERE `code` = '$code' ORDER BY `speclevel`,`buildtime` ASC";
    $stmt = $conn->query($sql);
    //匯出該代碼所有資料
    $allrows = $stmt->fetchAll(PDO::FETCH_ASSOC);
    //只取得相同階層之最新日期資料
    foreach ($allrows as $allrow) {
        $rows[$allrow["speclevel"]] = $allrow;
    }
    //計算階乘數
    foreach ($rows as $key=>$val) {
        $rows[$key]["cumproduct"] = array_product(array_slice(array_column($rows,"specqty"),0,$key));
    }
    //重新排序陣列
    krsort($rows);
    //開始計算數量
    if (intval($qty)<=0) {
        return strval($qty).array_slice($rows,-1)[0]["specname"];
    }
    $result = "";
    foreach ($rows as $key=>$val) {
        if ($key==count($rows)){
            $rows[$key]["quotient"] = intval($qty / $rows[$key]["cumproduct"]);
            $rows[$key]["remainder"] = $qty % $rows[$key]["cumproduct"];
            $prevremainder = $rows[$key]["remainder"];
            if ($rows[$key]["quotient"] > 0){
                $result .= number_format(strval($rows[$key]["quotient"])).strval($rows[$key]["specname"])." ";
            }
        }else{
            $rows[$key]["quotient"] = intval($prevremainder / $rows[$key]["cumproduct"]);
            $rows[$key]["remainder"] = $prevremainder % $rows[$key]["cumproduct"];
            $prevremainder = $rows[$key]["remainder"];
            if ($rows[$key]["quotient"] > 0){
                $result .= number_format(strval($rows[$key]["quotient"])).strval($rows[$key]["specname"])." ";
            }
        }
    }
    return trim($result);
}
```
實際上應用如下，範例圖片還有用了 bootstrap 的 tooltips 設計：
![PHP pkgcalc](php-pkgcalc.png#center)
***
## 資料表應用：大變小
程式上線後一段時間，藥品數量的部分逐漸變成包裝量，但有些時候還是需要顆數來向院內的系統進行溝通。因此就必須逆著把程式寫回來。
### PHP
將資料表中匯出後形成已經乘算好的陣列：
```php
$sql = "SELECT * FROM pkgspec WHERE `code` = '$code' ORDER BY `speclevel`,`buildtime` ASC";
$pkgspec_tmp = $conn->query($sql)->fetchAll(PDO::FETCH_ASSOC);
foreach ($pkgspec_tmp as $value) {
    $pkgspec[$value["speclevel"]] = $value;
}
foreach ($pkgspec as $key=>$value) {
    $pkgspec[$key]["cumproduct"] = array_product(array_slice(array_column($pkgspec,"specqty"),0,$key));
}
$pkgspec = array_column($pkgspec, "cumproduct", "specname");
```
已經輸入的規格量利用正規表示式切開，並擷取數字後帶回上述的陣列中計算：
```php
preg_match_all('/(\d+)('.implode("|",array_keys($pkgspec)).')/', $pkginput, $matches);
if (count($matches[0])>0){
    $quantity = 0;
    foreach ($matches[0] as $value) {
        $unit = preg_split("/(?<!\p{Han})(?=\p{Han})|(?<![0-9])(?=[0-9])/u", $value, 0, PREG_SPLIT_NO_EMPTY)[1];
        $quantity += preg_split("/(?<!\p{Han})(?=\p{Han})|(?<![0-9])(?=[0-9])/u", $value, 0, PREG_SPLIT_NO_EMPTY)[0] * $pkgspec[$unit];
    }
    if ($quantity>0) {
        return($quantity);
    }
}
```
### JavaScript
為了優化使用者體驗，不讓使用者送出表單再返回結果，所以直接把資料表用 PHP 轉成 json ：
```php
$sql = "SELECT `code`,`speclevel`,`specqty`,`specname` FROM pkgspec ORDER BY `speclevel`,`buildtime` ASC";
$allrows = $conn->query($sql)->fetchAll(PDO::FETCH_ASSOC);
//只取得相同代碼相同階層之最新日期資料
foreach ($allrows as $allrow) {
    $rows[$allrow["code"]][$allrow["speclevel"]] = $allrow;
}
//計算階乘數
foreach ($rows as $code=>$inners) {
    foreach ($inners as $key=>$val) {
        $rows[$code][$key]["cumproduct"] = array_product(array_slice(array_column($inners,"specqty"),0,$key));
        unset($rows[$code][$key]["code"]);
        unset($rows[$code][$key]["specqty"]);
        unset($rows[$code][$key]["speclevel"]);
        $unit[$code] .=  $val["specname"];
    }
    krsort($rows[$code]);
    $rows[$code] = array_values($rows[$code]);
}
//形成json傳給jQuery
$pkg = json_encode($rows, JSON_UNESCAPED_UNICODE);
$unit = json_encode($unit, JSON_UNESCAPED_UNICODE);
```
再讓 Javascript 去接 json ，如同剛剛的 PHP 一樣使用正規表示式切開字串：
```html
<script>
const pkg = <?= $pkg; ?>;
const unit = <?= $unit; ?>;

function pkgcalc(code, qty) {
    let legal = 0;
    let output = 0;
    const split_re = new RegExp("(\\d+["+unit[code]+"])");
    const match_re = new RegExp("["+unit[code]+"]");
    $.each(qty.split(split_re), function(i, e) {
        if ((e=="") || (e.slice(-1).match(match_re))) {
            legal += 0;
            if (e!="") {
                $.each(pkg[code], function(pi, pe) {
                    if (e.slice(-1) == pe.specname) {
                        output += parseInt(e) * pe.cumproduct;
                    }
                });
            }
        } else {
            legal += 1; 
        };
    });
    if (legal>0) {
        return "？";
    } else {
        return output;
    };
}
</script>
```
並且使用 jQuery ，讓表單值有變更的時候觸發函式：
```html
<script>
$(function(){            
    $("#drug").change(function(){
        $("#code").val("");
        $("#qty").val("");
        $("#result").html("？")
        $.each(drugs, function(index, element){
            if(element == $("#drug").val()){
                $("#code").val(index);
            };
        });
    });
    $("#qty").keyup(function () {
        const code = $("#code").val();
        const qty = $("#qty").val();
        $("#result").html(pkgcalc(code, qty));               
    });
});
</script>
```
結果如下：
![PkgCalc Tool](PkgCalc-Tool.gif#center)
***
## 資料表的修改
資料表的的修改，直接用 php 寫一頁 UI 表單配合 MySQL 簡單操作資料表即可。
![pkgcalc CRUD UI](pkgcalc-crud-ui.png#center)

比較困難的地方是要與**院內的藥品品項同步**。當有新增品項時，要隨著藥品種類帶入預設值，例如：咳嗽藥水的一級包裝名稱是「瓶」，針劑的一級包裝名稱是「支」等等。當院內品項被刪除時，原則上可以保留資料表中資料，只是不使用所以不影響。

因此該資料表必須每天和院內的藥品品項進行比對，如果資料有**差集**，必須判斷後新增預設值進資料表中。
```python
import pandas as pd
import mysql.connector
from sqlalchemy import create_engine
engine = create_engine('mysql+mysqlconnector://使用者名稱:密碼@伺服器存在位置IP:埠號/資料庫名稱')
df = pd.read_excel('院內系統匯出的藥品品項.xls', dtype=str)
pkgspec = pd.read_sql('SELECT DISTINCT `code` FROM `pkgspec`', con=engine)
```
兩個集合 `df` 、 `pkgspec` 的資料取單一邊的差集。
```python
new_insert = list(set(df['藥品代碼'].tolist()) - set(pkgspec['code'].tolist()))
pkgspec_time = pd.to_datetime('now').strftime('%Y%m%d%H%M%S')
```
將差集資料以預設值新增進資料表中，並且只先完成第一層級的資料。
```python
for code in new_insert:
    name = df.loc[df['藥品代碼']==code, '藥品名稱'].values[0]
    unit_zh = {'1KIU':'千單位','KU':'千單位','AMP':'支','BAG':'袋','BALL':'顆','BOT':'瓶','BOX':'盒','CAP':'顆','CART':'支','DOSE':'支','GM':'克','IU':'單位','MG':'毫克','ML':'毫升','PACK':'包','PATC':'片','PEN':'支','PIEC':'片','PIECE':'片','SUPP':'顆','SYRI':'支','TAB':'顆','TUBE':'支','U':'單位','VIAL':'支','KG':'公斤','SET':'組'}
    #院內醫師會有英文的開立處方單位，用一個 dict 轉換翻譯。
    df = df.replace({'院內設定包裝':unit_zh})
    specname = df.loc[df['藥品代碼']==code, '院內設定包裝'].values[0]
    engine.execute(f"INSERT INTO `pkgspec` (`id`, `code`, `name`, `specname`, `specqty`, `speclevel`, `buildtime`) VALUES (NULL, '{code}', '{name}', '{specname}', '1', '1', '{pkgspec_time}');")
```
如果不先新增預設值，上面寫的自訂函式碰到新的藥品代碼，會跳出錯誤造成程式中止。所以要嘛就是要在自訂函式中加入例外確認，避免錯誤中止程式；要嘛就是每天檢查新的藥品代碼。為了維持資料在最新的狀態，後來選擇了後者，畢竟如果寫了太多的 try 、 exception ，程式少了很多錯誤訊息，累積一段時間後才來維護也是挺麻煩的。
***
## 其他的應用實例
### 藥品盤點單的庫存數字
![PkgCalc Intracounts](intracounts.png)
### 調藥系統的數量輸入介面
![Transfer System](transfer-system.png#center)
當初會需要規格化藥品的包裝，主要是為了調藥系統的介面優化。初期調藥系統剛建立時，使用者需要輸入藥品的最小包裝量，以最上面的例子而言，如果使用者從藥庫調撥走 5 箱的 Bokey 膠囊，就要在調藥系統上輸入 58800 顆。

規格化藥品包裝後，發現該項功能無論是小換算大或是大換算小，都能夠減輕一些工作效率，也能夠減輕事務員的經驗仰賴，無論是人力輪調或人力訓練都提升了不少好處。
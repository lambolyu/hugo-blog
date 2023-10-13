+++
title = '藥品消耗量分析：前置作業'
slug = '2023-10-consumption-analyzation-preprocess'
date = 2023-10-10T08:00:40+08:00
draft = false
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['Data Visualization']
tags = ['Python','MySQL','Database','MariaDB','Pandas','DataFrame','資料庫','時間序列','time series','date_range','merge','groupby','Grouper','pivot']
+++

在[前幾篇](https://lambolyu.netlify.app/2023/09/medicine-consumption-data-visualization/)有提過，我們利用 python 彙整處方資料至資料庫中，製作視覺化圖表便於讓藥師進行決策。

但是這功能卻僅限於***已經知道特定藥品***的異常，輸入該藥品名稱，才會進入該藥品的視覺化圖表進而了解異常的區間或異常的情形。

要怎麼***已經知道***？醫院的藥品動輒上千種，以本院為例，藥庫需要掌握的藥品數量大概在 1,500 種上下，因此需要建立額外的方法**直接讓程式篩選**出異常藥品，另外跳提示。
***
## 藥品消耗資料 = 時間序列資料
比較穩定的藥品消耗資料屬於時間序列資料的一種，部分具穩定性 (stationary) 、季節性 (seasonality) 和自我相關 (autocorrelation) ，因此可以為每個藥的消耗資量建立簡單的數學模型以描述其波動狀態。

>**什麼是不穩定的藥品消耗資料？**
>
>包含零星個案的昂貴藥品、罕見疾病的治療藥物等使用的母群體數量不大，雖然偶有規律性，卻會因為個案或其他原因打斷規律性，導致資料不穩定。

但是醫院星期六只有上午看診或是星期日沒有看診，此時藥品的消耗量大幅降低，甚至沒有消耗，序列資料就會受到影響，這種影響在進入時間序列分析時，會被數學運算歸納成新的特徵，進而改變模型描述資訊。因為我們要將各藥品之間互相做比較，六日使用量低的特徵比較沒有意義，所以為了避免六日的干擾，我們會將整週的消耗量相加，直接利用週跟週之間的消耗量重新形成新的序列，~~類似降維的概念~~，以弭平六日的影響。

![Day to Week](daytoweek.png#center)
如上圖，上面兩張圖和下面兩張圖分別為不同藥品，可以觀察到上面的藥品容易受到六日的干擾，我們後續使用程式篩選出異常資料時，程式可能會判定其中一種情形為異常，例如如果大部分的藥品六日都沒有消耗，下面那種就會是異常，但其實六日波動對於藥品消耗不是異常情形，改為週總和序列後取消六日的差異，讓程式將重點放在其他異常情形上。
***
## 程式規劃
由於 php 計算相關的數學模型需要付出的效能以及程式大小已經大大影響使用者體驗，所以偏好將計算與警示的工作交給 python ：
1. 用 python 將資料匯出資料庫後利用函式庫進行科學計算。
2. 形成中繼檔案交由 php 讀取。
3. 中繼的檔案格式不限，但開發的時候考慮閱讀便利所以選用 csv 檔。
4. php 讀到資料後，在同一個網頁裡面丟給 chartjs 作圖。
![Alert program design](alert-design.png#center)

***
## 資料收集與整理
為了後續分析方便和操作資料比較直覺，我們將原始資料庫裡的資料作以下的整理：
1. 將每一筆消耗量以日期為群組進行相加，得到`日期：日消耗總和`的陣列或字典。
2. 將不連續的`日期：日消耗總和`資料補上連續日期資料，消耗數量空值補 0 。
3. 將日期重新取樣成每週，並對資料再次進行相加，得到`週序：週消耗總和`的陣列或字典。
4. 將資料改成下圖的形式，對於後續可讀性或操作回饋比較好。
![Alert Data Like](alert-data-like.png#center)

先匯出資料庫中的資料，需要決定一個起始日期，例如一年前的當月月初，然後一樣使用 MySQL GROUP BY 查詢返回 SUM 來計算總和：
```python
import pandas as pd
import numpy as np
import mysql.connector
from sqlalchemy import create_engine
engine = create_engine('mysql+mysqlconnector://~')

code = '藥品代碼'

# 起始日期，假如今天是 2023-09-22，則起始日期為 2022-09-01
start_day = (pd.to_datetime('today') - pd.tseries.offsets.MonthBegin(13)).strftime('%Y%m%d')
today = pd.to_datetime('today').strftime('%Y%m%d')

# 查詢藥品用量資料
capital = '0' if code[0].isnumeric() else code[0]
sql = f"SELECT `date`,SUM(`total`) AS `qty` FROM `consmp_{capital}` WHERE `drug` = '{code}' AND `date` >= '{startday}' GROUP BY `date`"
df = pd.read_sql(sql=sql, con=engine)
```
***
## pd.date_range
如同先前 php 的 `new DatePeriod()` 類別建立連續時間的 array ， pandas 則是利用 `.date_range()` 建立一個連續時間的 list 。例如以下的程式碼，就會建立一個以 2023 年 9 月 20 日 00 點 00 分 00 秒為第一個元素，間隔一天新增一個元素的的 DatetimeIndex 物件 ( python 中也可以視為 list 的一種) ：
```python
pd.DataFrame(pd.date_range(start='2023-09-20', end='2023-09-22', freq='D')
```
執行結果如下：
```powershell
DatetimeIndex(['2023-09-20 00:00:00', '2023-09-21 00:00:00', '2023-09-22 00:00:00'], dtype='datetime64[ns]', freq='D')
```
## pd.DataFrame
接著就可以把這個連續日期的 list 直接塞進 `.DataFrame()` 裡面，設定 `columns` 參數，變成一張具有連續日期的表格，但是因為 `.date_range()` 建立出來的字串是時間戳型別，所以要先用 `.strftime()` 強制轉型：
```python
pd.DataFrame(pd.date_range(start='2023-09-20', end='2023-09-22', freq='D').strftime('%Y%m%d'), columns=['date'])
```
執行結果如下：
```powershell
       date
0  20230920
1  20230921
2  20230922
```
## pd.merge
php 是利用 `foreach()` 直接迭代新的連續時間陣列，並將舊資料逐筆塞入。pandas 則是用 `.merge()` 方法把兩張表合併，我比較常用的參數有：
```python
新表 = pd.merge(左表, 右表,
    on = '以哪一直欄為索引',
    how = '合併方法'
)
```
也可以寫成：
```python
新表 = 左表.merge(右表,
    on = '以哪一直欄為索引',
    how = '合併方法'
)
```
`how` 是指定左表和右表的結合方式，預設值是 `inner` ，取**交集**索引和該索引的資料：
![Pandas Merge Inner](merge-inner.png#center)

參數給 `outer` 表示**聯集**，空出來的格子則會被塞入 `NaN` ：
![Pandas Merge Outer](merge-outer.png#center)

其他還有以左表為主的 `left` ，或是右表為主的 `right` ，都很常用：
![Pandas Merge Left](merge-left.png#center)
![Pandas Merge Right](merge-right.png#center)

## pd.groupby with pd.Grouper
連續時間資料的 DataFrame 與原本總和的 DataFrame 合併之後，資料會長這樣：
![Alert Data After Merge](data-day-aftermerge.png#center)

現在要把日期每七天加在一起，最直覺的可能就是用**切片**的方式，每七個的資料切一刀計算總和，但無論如何第一件要做的事情是：以藥名  **分組**。
```python
df = df.groupby('藥品代碼')
```
這個指令的結果會產生出一個 Pandas 的物件：
```powershell
<pandas.core.groupby.generic.DataFrameGroupBy object at 0x00000>
```
如果今天資料沒有日期，這個 Pandas 的物件後面就可以使用 `.sum()` 或是 `.mean()` 來求出分組結果，跟 MySQL 的指令一樣。

這麼一來會發現一開始想的切片方式比較難以在 Pandas 的物件裡面操作，後來決定使用 Pandas 內建的重新取樣功能  `.resample()` ，可以依照時間戳格式的欄位，將資料進行分組取樣。不過這個 `.resample()` 方法，必須讓時間戳欄位為**索引 index** ，所以要這樣寫：
```python
df = df.set_index('日期').groupby('藥品代碼').resample('取樣頻率')
```

關於取樣頻率可以指定的方式有[這些](https://pandas.pydata.org/docs/user_guide/timeseries.html#timeseries-offset-aliases)，我們以每週取樣，再將 `數量` 以總和的方式計算，所以寫成這樣：
```python
df = df.set_index('日期').groupby('藥品代碼').resample('W')['數量'].sum()
```

這行程式的寫法解釋起來很通順，卻不知道為什麼超級耗時間和效能，後來找到了一篇 [stackoverflow](https://stackoverflow.com/questions/51050157/why-is-resample-much-slower-than-pd-grouper-in-a-groupby) ，發現也有人有這個問題，雖然沒有解釋為什麼，但是提問者有提出另外一種寫法，不只簡單，效能也比較好：
```python
df = df.groupby(by=['藥品代碼', pd.Grouper(freq='W', key='日期')]).sum()
```

重新取樣之後資料會由星期一加到星期日，並且顯示為星期日的日期：
![Resample by week](data-resample.png#center)

但此時如果未滿一個禮拜，例如假設今天是禮拜三，重新取樣會將星期四、星期五、星期六和星期日的資料視為 0 進行加總，出現下星期日的日期，但這個資料點有太多的 0 了，並不是真實情形，所以會在後面使用其他程式排除。

## pd.pivot_table
![Alert Data After Maerge](data-week-aftermerge.png#center)
接著要把**週序**移到橫列當作 columns name ，需要用到 `pivot_table` 樞紐分析表的功能，下面列舉一些比較常用的參數：
```python
df = pd.pivot_table(
    DataFrames,
    values='',
    index='',
    columns='',
    aggfunc='mean'
)
```
`values` 、 `index` 、 `columns` 是必須欄位，要填入來源 DataFrame 的 columns 名稱，也可以上 list 形式變成兩級 index 。其中 `values` 為資料來源， `index` 為列 (row) 的來源， `columns` 為欄 (column) 的來源：
![Pandas DataFrame Pivot](pivot-demo-finish.png#center)

如果資料來源有衝突的時候，資料就以 `aggfunc` 的方式處理，預設為 `mean` 平均值，例如以下的情形就會取 300 和 400 的平均值為 350 ：
![Pandas DataFrame Pivot Conflict](pivot-demo-conflict.png#center)

>**其實還有 `pd.pivot` 可以使用！**
>
>差別在於 `.pivot()` 沒有 `aggfunc` 的參數，無法處理衝突值，等於是 `.pivot_table()` 的簡化版，如果確定資料沒有衝突，又不想多打幾個字，可以只用 `.pivot()` 就好。

## 程式實作
將以上的方法串聯在一起後，包成自訂函示，然後一樣以 `DataFrame.iterrows()` 遍歷藥品品項檔，最後完成整個樞紐分析表：
```python
import numpy as np
import pandas as pd
import mysql.connector
from sqlalchemy import create_engine

def consmp(drug):
    # 選定時間區間: 前一年的該月第一天起
    start_day = (pd.to_datetime('today') - pd.tseries.offsets.MonthBegin(13)).strftime('%Y%m%d')
    today = pd.to_datetime('today').strftime('%Y%m%d')
    # 查詢藥品用量資料
    capital = '0' if drug[0].isnumeric() else drug[0]
    sql = f"SELECT `date`,SUM(`total`) AS `qty` FROM `consmp_{capital}` WHERE `drug` = '{code}' AND `date` >= '{startday}' GROUP BY `date`"
    engine = create_engine('mysql+mysqlconnector://root:Medcine5640!@192.168.12.215:3307/consmp')
    # 將 sql 讀取成 df
    df = pd.read_sql(sql=sql, con=engine)
    df['藥品代碼'] = drug
    if len(df) :
        # 連續日期的 df
        dayrange = pd.DataFrame(pd.date_range(start=start_day, end=today, freq='D').strftime('%Y%m%d'), columns=['date'])
        df = pd.merge(dayrange, df, how='left', on='date').fillna(0)        
    else:
        # 如果 sql 讀不到資料，可能是該藥品在期間都沒有耗用
        df = pd.DataFrame(pd.date_range(start=start_day, end=today, freq='D').strftime('%Y%m%d'), columns=['date'])
        df['數量'] = 0
    
    df = df.groupby(by=['藥品代碼', pd.Grouper(freq='W', key='日期')]).sum()
    # 未滿一整週的資料排除：如果昨天不是星期日，就去掉最後一個週序資料
    yesterday = (pd.to_datetime('today') - pd.tseries.offsets.DateOffset(days=1)).strftime('%Y%m%d')
    if df.columns[-1] != yesterday:
        df = week.iloc[:,0:-1]
    return df

# 遍歷藥品品項檔輸出 df
med = pd.read_csv('藥品品項檔.csv')
dflist = []
for _,row in med.iterrows():
    dflist.append(consmp(row['藥品代碼']))

# 利用 concat 合併 df
df = pd.concat(dflist, ignore_index = False)

```

完成！這個 df 就是我們所需要的表格。先以 csv 檔或是其他檔案類型暫存起來，下一篇就要開始正式進行分析跟篩選：
```python
df.to_csv('暫存.csv', index=False)
```

***
## Attribution
- [Icons created by Freepik - Flaticon](https://www.flaticon.com/)
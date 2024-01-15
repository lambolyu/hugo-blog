+++
title = '藥品採購需求預測：簡單數學模型'
slug = 'Medicine-purchasing-forecast-with-simple-math-model'
date = 2024-02-07T09:58:36+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = []
tags = []
+++
上一篇有說明醫院產生藥品採購單的方法，總結一下我們需要的參數有：
- 目前庫存量
- 已經採購的在途量
- 安全量
- 採購基準量
有了這些參數之後，我們就可以產生**現在**的藥品採購單。

那有沒有辦法產生**未來的**藥品採購單呢？
***
## 未來的採購需求
現在的採購需求是來自於下面的邏輯：

如果目前庫存量 + 在途量 < 安全量，則計算 [( 基準量 - 庫存量 ) // 基準量 ] × 基準量
>**//** 表示相除後得到商數和餘數，只取商數，例如 10//3=3 ，但有些時候我會相除之後讓商數呈現小數點，再將商數四捨五入取整數。

也就是說，如果我們需要未來的採購需求，應該是把邏輯變成：

如果**未來**庫存量 + 在途量 < 安全量，則計算 [( 基準量 - 庫存量 ) // 基準量 ] × 基準量

而未來的庫存量其實就是目前的庫存量 - 未來的消耗量，假設今天是 2 月 7 日，如果我想知道 3 月 7 日的庫存量，就是把目前的庫存量 - 各個藥品 28 天的消耗量就可以了。

所以下一個問題是，未來消耗量怎麼計算？
***
## 時間序列預測
藥品的消耗資料先前已經被整理成每日統計的形式，可以被視為一組時間序列 (Time Series) ，以本院的 Plavix 75mg Tablet 為例，將日期當作 x 軸，消耗量當成 y 軸，就可以簡單畫出下面的折線圖。
![timeseries](timeseries.png)

時間序列預測未來的數值是這幾年來的顯學之一，歷經了使用機器學習法的 ARIMA ，到如今深度學習大爆發時代的 LSTM 、 GRU 等等大家都致力發展這一塊，其中一個最夯的時間序列大概就是股票了吧，雖然股票的變因太多導致 AI 學習的成果通常都有限。

深度學習的部份，因為受限於醫院硬體，速度與效能的部分都需要多方的調整，這個日後找機會再談。這邊先簡單以傳統數學模型介紹現行使用的未來消耗量的預測。

### 傳統數學模型
傳統數學用來預測未來的數值的方法其實就是平均值和標準差 (Standard Deviation) ，以下面的資料為例：
```csv
date,consumption
20231106,9594
20231107,7266
20231108,8924
20231109,8917
20231110,7923
20231111,1906
20231112,143
20231113,7383
20231114,7721
20231115,9058
20231116,5462
20231117,6779
20231118,1792
20231119,162
20231120,7196
20231121,8528
20231122,9675
20231123,6170
20231124,5822
20231125,2478
20231126,109
20231127,10432
20231128,8767
20231129,7174
20231130,8428
20231201,6561
20231202,1323
20231203,71
```

可以求得平均值為 5920 ， 標準差為 3346 ，因此我們可以判斷 2023 年 12 月 4 日的消耗量可能為 5920 ± 3346 = 9266 ~ 2574 之間。

這樣的數字其實沒什麼意義，因為忽略了一些大前提，我們以平均值和標準差來預測未來的數值時，母體數值群 ( 也就是被用來計算的過去資料 ) 必須呈現常態分佈，並且資料必須平穩集中。

我們引入整年份的資料，並利用 `Shapiro-Wilk test` 來對資料進行常態分佈的檢定：
```python
import pandas as pd
import scipy.stats

df = pd.read_csv('整年度某個藥品的資料.csv')
tests = scipy.stats.shapiro(df['consumption'].tolist())
```

執行後可以得到這個結果：
```powershell
ShapiroResult(statistic=0.8714791536331177, pvalue=6.371240215925978e-17)
```
`Shapiro-Wilk test` 的虛無假設 H0 是資料屬於常態分佈，如果結果的 pvalue 小於 0.05 ，則會拒絕虛無假設；以我們的資料來說， pvalue = 6.371240215925978e-17 遠小於 0.05 ，因此我們可以確定這個資料不屬於常態分佈，當然以平均值和標準差來預測未來的數值就會出現離譜的偏差。

### 資料的降採樣

那下一步該怎麼做？

藉由剛剛的圖形稍微觀察到，醫院藥品的耗用情形大致是以每七天的一周期的方式波動，禮拜六日因為醫師休診醫院放假，耗用量就幾乎為零，因此我們可以以每七天重新對資料採樣一次，大概有點類似降採樣 (Down Sampling) 的概念。

於是我們可以將剛剛的資料在多加一行 (column) 的星期幾的資料，如此就可以利用 `.loc()` 將特定星期的資料分出來：
```python
df['weekday'] = pd.to_datetime(df['date'], format='%Y%m%d').dt.strftime('%w')
# 0=星期日, 1=星期一, 2=星期二 ..., 6=星期六

df_w0 = df.loc[df['weekday']=='0']
...
df_w6 = df.loc[df['weekday']=='6']
```

我們來看看每七天重新對資料採樣一次的 `Shapiro-Wilk test` 的 pvalue 有沒有下降：
```python
for i in range(0, 7):
    print(f'w{i}: ', end='')
    w = df.loc[df['weekday']==str(i)]
    tests_w = scipy.stats.shapiro(w['consumption'].tolist())
    print(tests_w)
```

結果為：
```powershell
w0: ShapiroResult(statistic=0.9827414155006409, pvalue=0.6361146569252014)
w1: ShapiroResult(statistic=0.8977280259132385, pvalue=0.00030909766792319715)
w2: ShapiroResult(statistic=0.7871079444885254, pvalue=3.008760245393205e-07)
w3: ShapiroResult(statistic=0.7868373394012451, pvalue=2.966743011256767e-07)
w4: ShapiroResult(statistic=0.816359281539917, pvalue=1.225354139933188e-06)
w5: ShapiroResult(statistic=0.7929207682609558, pvalue=3.3607966543058865e-07)
w6: ShapiroResult(statistic=0.9263978004455566, pvalue=0.002929063281044364)
```

可以看到 pvalue 明顯大了許多，甚至禮拜日 w0 的資料已經屬於常態分佈了。

於是我接著將資料由一年份開始，檢查 `Shapiro-Wilk test` 結果，如果不屬於常態分佈，就將頭七天扣掉，以此將資料逼近現在的時間，這麼一來我就可以得到資料從第幾週的範圍開始選取會是常態分布：
```python
result = {}
for i in range(0, 7):
    df = pd.read_csv('整年度某個藥品的資料.csv')
    df['weekday'] = pd.to_datetime(df['date'], format='%Y%m%d').dt.strftime('%w')
    n = 52
    while True:
        df = df.iloc[-7*n:,:]
        w = df.loc[df['weekday']==str(i)]
        tests_w = scipy.stats.shapiro(w['consumption'].tolist())
        pvalue = tests_w.pvalue
        if (pvalue <= 0.05) & (n>0):
            n -= 1
        else:
            result[f'w{i}'] = n
            break
```

並且結合之前說過的平均值和標準差來對未來資料進行預測，這邊我使用平均值加上 0.1 倍的標準差，消耗量的資料寧可高估也不要低估：
```python
result = {}
for i in range(0, 7):
    df = pd.read_csv('整年度某個藥品的資料.csv')
    df['weekday'] = pd.to_datetime(df['date'], format='%Y%m%d').dt.strftime('%w')
    n = 52
    while True:
        df = df.iloc[-7*n:,:]
        w = df.loc[df['weekday']==str(i)]
        tests_w = scipy.stats.shapiro(w['consumption'].tolist())
        pvalue = tests_w.pvalue
        if (pvalue <= 0.05) & (n>0):
            n -= 1
        else:
            #result[f'w{i}'] = n
            result[f'w{i}'] = w['consumption'].mean() + 0.1*w['consumption'].std(ddof=0)
            break
```

結果為：
```powershell
{'w0': 140.89182907442253,
 'w1': 8515.939732601033,
 'w2': 8195.225040738227,
 'w3': 8879.406632279424,
 'w4': 7250.917852989266,
 'w5': 7158.895150689425,
 'w6': 1871.8293178917595}
```

這樣就可以利用簡單的數學模型計算每個 weekday 的用量。

***
## 程式流程規劃與實作
***
## 成果
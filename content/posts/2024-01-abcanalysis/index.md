+++
title = '庫存管理 ABC 分析法'
slug = '2024-01-abc-analysis'
date = 2024-01-15T08:22:39+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['python']
tags = []
+++
## ABC 分析法是什麼
***







ABC 分類法，將藥品以年度消耗的總金額排序後累加後排序，單價比較高但是消耗比較少的稱為 A 類品項，單價比較低而且消耗比較多的成為 C 類品項，將每個藥品定義好 ABC 的類別後，再依 ABC 去定義各自的請購點庫存。
![ABC analysis](https://www.researchgate.net/profile/Hooshang-Beheshti/publication/256852552/figure/fig1/AS:340619374415883@1458221395785/Pareto-chart-of-ABC-classification-of-coagulation-and-hematology-reagents.png#center)

　

執行 ABC 分析法的步驟如下：
1. 計算每個藥品一年度的消耗量總和
2. 找出藥品的單價
3. 由上面兩點計算出每個藥品一年度的消耗總金額
4. 將每個藥品一年度的消耗總金額依照大到小降冪排序
5. 計算每個藥品的累加消耗金額
6. 計算所有藥品的總消耗金額
7. 計算每個藥品的累加消耗金額比率 = 累加消耗金額 / 總消耗金額
8. 訂定 ABC 的分界

>2*. 藥品的單價，如果可以找到成本價更好，但是醫院藥局端無法得知成本價，只好用健保價或自費價

>8*. ABC 的分界，每個公司的標準都不太一樣

　

程式的部分沿用先前已經建立好的資料庫資料：
```python
import pandas as pd
import numpy as np
import mysql.connector
from sqlalchemy import create_engine
engine = create_engine('mysql+mysqlconnector://帳號:密碼@位置:埠號/資料表名稱')

# 1. 計算每個藥品一年度的消耗量總和
start_date = (pd.Timestamp('today') - pd.tseries.offsets.MonthBegin(13)).strftime('%Y%m%d')
end_date = (pd.Timestamp('today') - pd.tseries.offsets.Day(1)).strftime('%Y%m%d')

df = pd.read_excel('藥品基本檔.xls')['藥品代碼'].tolist()
for code in df:
    capital = '0' if code[0].isnumeric() else code[0]
    sql = f"SELECT `drug`, SUM(`total`) AS `qty` FROM `consmp_{capital}` WHERE `drug` = '{code}' AND `date` >= '{start_date}'"
    df = pd.read_sql(sql, engine).rename(columns={'drug':'藥品代碼','qty':'年用量'})
df = pd.concat(dflist, axis=0, ignore_index=True).fillna(0)    

# 2. 找出藥品的單價
bs = pd.read_excel('藥品基本檔.xls')[['藥品代碼','自費價','健保價']]
bs['價格'] = bs['健保價']
bs.loc[bs['健保價']=='0', '價格'] = bs['自費價']
bs = bs[['藥品代碼','價格']]
df = pd.merge(bs, df, on='藥品代碼', how='right')

# 3. 計算出每個藥品一年度的消耗總金額
df['消耗金額'] = df['年用量'] * df['價格']

# 4. 將每個藥品一年度的消耗總金額依照大到小降冪排序
df = df.sort_values(by=['消耗金額'], ascending=False)
```

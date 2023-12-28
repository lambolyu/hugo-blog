+++
title = '藥品採購需求預測：傳統數學模型'
slug = 'Medicine-purchasing-forecast'
date = 2024-01-27T08:22:39+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['Python','PHP']
tags = []
+++
## 庫存管理中的存貨模式
存貨模式指的是貨品在單位內的存量管理模式，沒有賣掉的貨品金額是最直接的庫存成本，而單位的首要目標就是降低庫存成本，所以最理想的狀況是維持零庫存，但如此一來又會增加物流金額，這也是另一種有形的庫存成本，而且在零庫存的狀態下，無疑會增加超賣或是賣完的銷售壓力──對於醫院來說可能是會直接影響到病人生命的事情。

因此目前最理想的狀態是，倉儲內維持必須維持一定的庫存量，並且依照物流成本減少訂購的次數。現行的訂購法分成兩種模式：定期訂購法 (Periodic Order Method) 和定量訂購法 (Quantitative Order Method) 。

### 定期訂購法 (Periodic Order Method)
{{< figure src="https://slideplayer.com/slide/17656947/105/images/21/Periodic+Review+System+%28P%29.jpg" width="80%" alt="Periodic Order Method" align="center" >}}

定期訂購法的概念是：倉管人員或是電腦會固定一個週期發出採購請求，並且一次買**到**規定的某個量。以上圖為例，藍色的線表示目前庫存量，當固定的一段時間 P ，就會發出購買**到** T 庫存的需求量，也就是每次採購的量會隨著當時庫存改變，第一次採購的量是 T - IP1 ，第二次則是 T - IP2 ，第三次是 T - IP3 。

### 定量訂購法 (Quantitative Order Method)
{{< figure src="https://www.researchgate.net/profile/Dejan-Dragan-2/publication/343797726/figure/fig4/AS:927066249785346@1598041226890/Typical-appearance-of-the-continuous-review-system-of-the-inventory-control-in-a-steady.png" width="80%" alt="Quantitative Order Method" align="center" >}}

而定量訂購法則是會設定一個固定的請購點 (Reorder Point) ，如果目前的即時庫存少於請購點庫存時，就會發出採購請求，而此採購量也會是一個固定的量，每次採購會改變的就是時間間隔了，如上圖藍色的線表示目前庫存量，可以看到買次採購的時間間隔 T1 、 T2 、 T3 都不一樣。

而一般庫存管理模式認為定量訂購法相對於定期訂購法的優勢比較大，除了比較難以克服的電腦技術成本 (其實現在看起來門檻降低了不少) ，定量訂購法可以大幅降低庫存成本，因為每次採購量是固定的，甚至可以經過計算後稱為經濟訂單量 (Economic Ordering Quantity)。

>延伸閱讀：[經濟訂單量 Q* = (2CD/H)^(1/2) 的公式推導](https://zh.wikipedia.org/zh-tw/%E7%B6%93%E6%BF%9F%E8%A8%82%E5%96%AE%E9%87%8F)。

***
## 定量訂購法中的參數設置
因此本院藥庫採行的訂購模式是以定量訂購法為主，而定量訂購法有兩個重要的參數需要設置，分別是請購點庫存與固定訂單量。

***
## 採購量的形成
***
## 未來採購量的預測
***
## 網頁介面設計

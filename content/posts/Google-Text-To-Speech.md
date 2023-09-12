+++
title = 'Google Text to Speech'
slug = 'google-Text-to-speech'
date = 2023-09-12T15:28:57+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['工具小零件']
tags = ['Python','Google','TexttoSpeech','文字轉語音','語音合成','說話']
+++
什麼？！這個跟藥庫管理有關係嗎？！

## 緣起
一開始只是因為常常接到藥師不諳藥品英文發音的電話，導致彼此溝通不良，因此決定使用 Google Translation 的發音功能以正自我視聽。

>有別於歐美的藥名發音，重音通常是在第一音節；臺灣人的藥名發音，重音幾乎是落在第二音節，有一種日式發音的感覺。無論如何，語言是溝通的工具，彼此聽得懂就好，不用特別苛求。

後來因為調藥系統的建立，就有著讓電腦讀出正在被調動的藥品名稱的想法。除了可以建立操作上的回饋感，也可以讓藥庫的藥師即時接收到正在調動的資訊。
***
## 下載 Google Translation 的聲音
其實正確來說，我們是要利用 Google Text to Speech 文字轉語音或語音合成的服務。 Google 的[產品說明頁面](https://cloud.google.com/text-to-speech?hl=zh-tw#section-2)有提供範例可以試聽聲音。

這項服務是被 Google 納入 Google Cloud Platform 的商品之一，我們使用的情形是將固定的語音下載之後重複使用，所以並不會超過目前的免費額度。

>WaveNet 語音每月前 100 萬個字元免費。如果是標準 (非 WaveNet) 語音，則每月前 400 萬個字元免費。

如果有興趣，或是想要使用更流暢的語音合成，可以參考官方的[計價說明](https://cloud.google.com/text-to-speech/pricing?hl=zh-tw)。
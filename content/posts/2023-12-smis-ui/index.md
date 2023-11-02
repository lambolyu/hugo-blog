+++
title = '智慧防疫物資管理系統：介面'
slug = '2023-12-smis-ui'
date = 2023-12-26T13:07:58+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['PHP','Javascript']
tags = []
+++
這個月分別建立了關於智慧防疫物資管理系統的資料比對與通報檔案生成的功能，剩下的就是製作一個可以讓使用者互動的介面了！

這個介面為了讓各台電腦不用安裝執行檔，決定採用 web 的方式呈現，所以就使用 php 做為後端資料庫溝通、後端 csv 檔案讀取，加上一點少少的 JavaScript 強化使用者體驗。
***
## 網頁切版
為了求製作快速，我使用了慣用的 [Bootstrap](https://getbootstrap.com/) 簡單用裡面的 grid 功能進行網頁切版，規劃的概念圖如下：
{{< figure src="div-design.png" width="65%" alt="page-block-design" align="center" >}}
***
## 確認 csv 更新了沒
***
## 院內剩餘庫存
***
## 待通報區域
***
## 手動通報
***
## 已通報區域
***
## ~~使用趨勢圖表~~
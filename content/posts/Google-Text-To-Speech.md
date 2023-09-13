+++
title = 'Google Text to Speech + Python'
slug = 'google-Text-to-speech-with-python'
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

這項服務是被 Google 納入 Google Cloud Platform 的商品之一， Google 每個月提供了一個免費的額度，超過額度之後就要開始依照字元收費。為了不超過免費額度，我們必須將固定的藥品名稱形成語音檔後下載回本地端使用，因為調藥系統無法連外網，二來這樣也不會佔掉免費額度。

>WaveNet 語音每月前 100 萬個字元免費。如果是標準 (非 WaveNet) 語音，則每月前 400 萬個字元免費。

如果有興趣，或是想要使用更流暢的語音合成，可以參考官方的[計價說明](https://cloud.google.com/text-to-speech/pricing?hl=zh-tw)。

## 開啟 Google Cloud 服務

首先登入 google 的帳號，並且打開 [Google Cloud](https://cloud.google.com/) 的頁面，並且點選右上角的免費試用。
![Google Cloud Activation](/images/2023-09-gcloud-01.png#center)

>**為什麼是試用？**
>
>就像剛剛提到的， Google Cloud 的服務其實是收費的，但是有提供每個月免費額度。只是因為有部分服務是收費的，所以啟用 Google Cloud 的時候要填寫信用卡資料。

接著填寫啟用資料：
![Google Cloud Activation](/images/2023-09-gcloud-02.png#center)

帳戶類型可以選擇個人，並填入個人信用卡資訊：
![Google Cloud Activation](/images/2023-09-gcloud-03.png#center)

啟用完成後，再填寫一些問題， Google Cloud 目前有提供 300 元美元的免費抵免額：
![$300 in free credits for new customers](/images/2023-09-gcloud-04.png#center)

正式進入頁面後，左上角有預設的專案 (project) ，建立不同的專案是為了處理多個團隊工作內容及團隊底下使用者的管理權限。但是因為我們使用 google Text to Speech 只是為了個人使用，不會超過免費額度，專案名稱是什麼並不會在任何地方顯示出來，所以可以直接用預設值 My First Project 就好。
![Google Cloud Projects](/images/2023-09-gcloud-05.png#center)

如果真的需要建立專案，也是可以點進左上角，再點選建立專案：
![Google Cloud Create Projects](/images/2023-09-gcloud-06.png#center)

就可自己建立一個比較喜歡的名稱，至於機構是什麼我就沒有特別去研究：
![Google Cloud Create Projects](/images/2023-09-gcloud-07.png#center)
***
## 啟用 Text to Speech 的 API
進入 Google Cloud 的[控制台](https://console.cloud.google.com/) (Console)，也就是剛剛可以看到左上角有預設專案名稱的那個頁面，點按左上角的 `≡` ，選擇 「 API 和服務」 ，再選擇 「程式庫」 ：
![Google Cloud API](/images/2023-09-gcloud-08.png#center)

在搜尋 API 和服務的欄位中輸入 「 text to speech 」 ：
![Google Cloud API Search](/images/2023-09-gcloud-10.png#center)

搜尋結果中選擇 Cloud Text-to-Speech API ：
![Google Cloud API Search](/images/2023-09-gcloud-11.png#center)

並點選啟用按鈕：
![Google Cloud API Search](/images/2023-09-gcloud-12.png#center)
***
## API 憑證
啟用完成後，點選左側選單中的 「 憑證 」 選項：
![Google Cloud API Search](/images/2023-09-gcloud-13.png#center)
接著在上面點選 「 + 建立憑證 」：
![Google Cloud API Search](/images/2023-09-gcloud-14.png#center)
並以 「 服務帳戶 」 的類型建立 API 憑證：
![Google Cloud API Search](/images/2023-09-gcloud-15.png#center)
這邊會需要註冊一個服務帳戶的名稱，這個名稱在個人使用的情境下一樣不會對外顯示，隨便取一個例如 「 tts 」 即可，接著可以跳過所有的表單空格，直接按下 「 完成 」 即可：
![Google Cloud API Search](/images/2023-09-gcloud-16.png#center)
***
## API 金鑰
建立完成後，點擊剛剛完成的服務帳戶：
![Google Cloud API Search](/images/2023-09-gcloud-17.png#center)
上方選單點選 「 金鑰 」 ：
![Google Cloud API Search](/images/2023-09-gcloud-18.png#center)
頁面中點選 「 新增金鑰 」 並選擇 「 建立新的金鑰 」：
![Google Cloud API Search](/images/2023-09-gcloud-19.png#center)
預設會以 json 檔案的形式建立，直接按下 「 建立 」 即可：
![Google Cloud API Search](/images/2023-09-gcloud-21.png#center)
接著瀏覽器就會直接下載建立好的 json 檔，請儲存在一個固定的位置：
![Google Cloud API Search](/images/2023-09-gcloud-22.png#center)
值得一提的是，這個 json 金鑰的期限很長，裡面會有很龐大的亂碼跟參數，並且**無法重新下載**，所以如果 json 檔案遺失了，只能刪除金鑰後再重新建立一次。
>**無法重新下載金鑰，只能刪除。**
>![Google Cloud API Search](/images/2023-09-gcloud-23.png#center)
***
## Python 與 Text to Speech API
先上 google 的[說明文件](https://cloud.google.com/python/docs/reference/texttospeech/latest)，該份文件只有英文~~我看的懂~~。

回到程式端，我們需要用 python 串接 API 以產生聲音檔，所以先下載用戶端的函式庫。這個指令會把 google-cloud-texttospeech 函式庫和相依的函式庫一起安裝好。~~滿多的。~~
```powershell
pip install --upgrade google-cloud-texttospeech
```

然後把剛剛的 json 絕對位置找出來，在程式裡面利用 `os` 方法加進系統的環境變數中：
```python
import os
os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = 'json 的位置'
```

引入 google 的函式庫：
```python
import google.cloud.texttospeech as tts
```

輸入需要發音的文字，要留意有沒有包含中文。其中參數 text 是使用一般方法建立語音，另外一種參數 ssml 則可以引入利用 xml 撰寫的語音標記來合成。 ssml 的說明如[連結](https://learn.microsoft.com/zh-tw/azure/ai-services/speech-service/speech-synthesis-markup-structure)，只是 ssml 的使用因為所有空白跟標記都算在額度之中，所以要謹慎使用。
```python
text = tts.SynthesisInput(text='Hello World')
```

選擇發音的語言和語音種類。[這邊](https://cloud.google.com/text-to-speech/docs/voices)有目前可選擇的 `language_code` 和 `name` ，選擇之前也可以先試聽看看。如果選擇 Standard 以外的聲音，可要稍微留意一下[免費額度](https://cloud.google.com/text-to-speech/pricing)。
```python
params = tts.VoiceSelectionParams(language_code='en-US', name='en-US-Wavenet-D')
```

其他設定，包含指定的輸出檔案類型 `audio_encoding` 、語速 `speaking_rate` 、音調 `pitch` 、音量增益 `volume_gain_db` 、合成採樣率 `sample_rate_hertz` 等等，都可以自己調整。相關的說明在[這裡](https://cloud.google.com/text-to-speech/docs/reference/rest/v1/AudioConfig)，以下是我自己習慣的設定：
```python
config = tts.AudioConfig(audio_encoding=tts.AudioEncoding.MP3, speaking_rate=1.5)
```

接著就把聲音合成出來，利用文件寫入的方式下載檔案：
```python
client = tts.TextToSpeechClient()
response = client.synthesize_speech(input=input, voice=params, audio_config=config)
with open('儲存的檔名.mp3', 'wb') as mp3:
    mp3.write(response.audio_content)
```
幾行之內就完成聲音的下載了，十分簡便！
***
## Python 聲音的操作
然而下載了聲音檔，直接拿來使用後發現：
- 英文語音預設語速過慢，語速要調整成 1.5 。
- 要加入中文語音避免自己也聽不懂英文藥名。
- 中文語音預設語速過慢，語速要調整成 0.9 。

因為語速和語調的不同，中英文必須分開合成下載，並且利用 python 把兩個聲音檔合併起來。此時我們需要另外一套函式庫幫助我們操作聲音。
```powershell
pip install pydub
```

***
## 利用 DataFrames 批次輸出語音
***
## 加碼：台語
+++
title = 'Windows Python 安裝與開發環境'
slug = '2023-11-my-python-environment'
date = 2023-11-16T08:21:46+08:00
draft = true
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['Python']
tags = []
+++
來記錄一下我的開發環境好了。開始學習並使用 Python 是 2017 年，不知道是看了什麼書或教學的影響還是根本沒看書，我自己的開發環境跟大家推薦的入門教學大相逕庭，而且因此踩了不少的坑，不過我並不討厭我現在開發模式，好像能比較較清楚來龍去脈。

但還有一個原因是，到目前為止我寫出來的程式都是個人開發個人使用，全部都是拿來解決工作上的小事，都不是團隊，所以可以讓我這麼恣意妄為。
***
## Python 的安裝
網路上的 python 教學為了各種友善新手的目的，會直接安裝 [Anaconda](https://www.anaconda.com/) ，或甚至會直接使用 [Google Colab](https://colab.research.google.com/) ，以此來省下建立環境、下載套件等等後面稍微複雜一點的步驟，以此讓新手的開發環境立刻達到和書上或是網路上的環境一樣的狀態。

先說，並不是不好。只是我個人喜歡原生的版本，所以還是選擇了從 [Python 的官方網站](https://www.python.org/)下載了 Windows 的安裝 exe 回來使用，版本當然是越新越好。

　

下載安裝檔並且執行之後有幾個選項要稍微留意一下，免得之後補救會有點複雜：
- 最底下的 Add python.exe to PATH **必須打勾**！
- 按下 Customize installation 自訂安裝目錄。
![python-install](python-install-1.png#center)

>預設安裝目錄放在 AppData 裡面，埋得很深，如果日後需要 python 的安裝目錄，可能就不太容易。因此選擇 Customize installation 在後面的步驟更改安裝目錄。

>Add python.exe to PATH 是將 python 加進 Windows 的環境變數中，意思是如果之後開啟 cmd 或是 PowerShell 等 CLI 視窗，輸入 python 指令即可開啟或執行，不需要多輸入 `C:\安裝目錄\python.exe`。

　

第二步：
- 不需要安裝 tcl/tk 和 IDLE ，所以將預設的勾拿掉。
- py launcher 和 for all users 可以打勾。
![python-install](python-install-2.png#center)

　

第三步，我習慣上將安裝目錄放在 `C:\Python+版本號碼\` 下面：
![python-install](python-install-3.png#center)

　

然後就是安裝並且完成。
![python-install](python-install-4.png#center)
![python-install](python-install-5.png#center)
***
## Python 套件更新
完成 python 的安裝之後，請先打開 cmd 或 PowerShell 的視窗，輸入 `pip list` ：
![pip-list](pip-1.png#center)

`pip` 指令是安裝 python 套件的指令。如果輸入 `pip list` 就是列出目前已經安裝的套件名稱。

新安裝 python 的環境，可以看到目前的套件只有 `pip` 自己，底下的 `[notice]` 顯示目前的 `pip` 已經是舊版本了，可以利用指令 `python -m pip install --upgrade pip` 來更新。
![pip-list](pip-2.png#center)

　

日後如果需要安裝或是更新套件，就直接利用 `pip install 套件名稱` 直接安裝至電腦裡就好，如果安裝這個套件還需要其他的套件，例如安裝 Pandas 時必須同時安裝 Numpy ，只要鍵入  `pip install pandas` 程式就會自動連 Numpy 一併安裝進去。
![pip-list](pip-3.png#center)
>我們稱這兩個套件具有相依 (dependency) 的關係，而且 Pandas 的相依套件也不只 Numpy 。

　

![Pypi](https://pypi.org/static/images/logo-large.9f732b5f.svg#center)
如果不知道套件名稱是什麼，例如看到程式中有一行 `import mysql.connector` 想要安裝此套件，但是 `pip install mysql` 卻怎麼裝怎麼錯，此時可以 Google `pypi mysql.connector` ，並且開啟 [pypi](https://pypi.org/) 網站的結果，這個網站就是 Python 套件的目錄，只要在這個目錄裡的套件，都可以使用 `pip` 下載安裝。
***
## Visual Studio Code
{{< figure src="https://notepad-plus-plus.org/images/logo.svg" width="20%" alt="notepad-plus-plus" align="center" >}}

撰寫程式的工具一開始是使用 [Notepad++](https://notepad-plus-plus.org/) ，後來因為看了很多工程師 Youtuber 各種神奇的操作：輸入短短的兩個字就自動寫滿整篇程式、各種五顏六色的文字顏色區隔等等，當年懵懂無知的我看到各種羨慕，就是就跳槽了。

　

{{< figure src="https://code.visualstudio.com/assets/images/code-stable.png" width="20%" alt="vscode" align="center" >}}

後來選擇的編輯器是 Microsoft 的 [Visual Studio Code](https://code.visualstudio.com/)，免費，使用了五六年到現在也堪稱穩定。甚至因為是 Microsoft 開發的，直接到官方網站下載安裝就可以直接使用了。

　

安裝過程中，相較於 python 日後可能異動或有版本更新等情形，編輯器相對穩定，因此我不會調整預設安裝目錄：
![vscode-install](vscode-install-1.png#center)
![vscode-install](vscode-install-2.png#center)

　

不喜歡程式在開始功能表和桌面上建立圖示：
![vscode-install](vscode-install-3.png#center)

　

另外我會把 「以 code 開啟」 的這兩個選項打勾：
![vscode-install](vscode-install-4.png#center)

　

這兩個選項打勾之後，對桌面或資料夾空白處按下右鍵，可以將指定的位置使用 VS code 開啟成工作目錄：
![vscode-install](vscode-install-5.png#center)

　

也可以直接對檔案按下右鍵，以 VS code 開啟檔案：
![vscode-install](vscode-install-6.png#center)

　

不過開啟之後，中文字體預設是新細明體，令我不太能接受，進入`檔案→喜好設定→設定`可以更改預設的設定，讓眼睛舒服一點。關於其他設定可以自行 Google 看看大家是怎麼調整自己的 VScode ，設定一個 codding 起來比較爽的介面。也可以直接點選設定視窗中的右上角的按鈕，把 vscode 的 `settings.json` 叫出來調整：
![vscode-settings-json](vscode-settings-json.png#center)

附上我個人的設定值供參考：
```json
{
    "editor.tabCompletion": "on",
    "editor.formatOnPaste": true,
    "editor.wordWrap": "on",//視區寬度類這行
    "editor.fontSize": 15,//編輯器字體大小
    "editor.fontFamily": "'Fira Code', 'Source Code Pro', '微軟正黑體', Consolas, 'Courier New', monospace",//在編輯器中使用字體
    "editor.fontLigatures": true,//啟用字體連字
    "editor.letterSpacing": -0.5,
    "editor.lineHeight": 25,
    "python.formatting.autopep8Path": "",
    "python.formatting.blackPath": "",
    "python.formatting.provider": "black",
    "python.formatting.yapfPath": "",
    "python.analysis.logLevel": "Information",
    "[python]": {
        "editor.formatOnPaste": false,
        "editor.formatOnType": true
    },
    "editor.unicodeHighlight.ambiguousCharacters": false,
    "editor.minimap.enabled": false,
    "workbench.startupEditor": "none",
    "git.confirmSync": false,
    "python.createEnvironment.trigger": "off",
}
```
***
## 再來呢？怎麼開始寫程式？
安裝好 Python 和 VScode 後就可以直接來試試看，先在桌面或是一個自訂的地方新增一個資料夾，例如我在桌面上建立 demo 的空白資料夾，並且打開他：
![hello world](hello-world-1.png#center)

　

右鍵叫出選單後點選 「以 Code 開啟」 ：
![hello world](hello-world-2.png#center)

　

打開的 VS code 視窗大概長這樣，不過依據每個人調整的情形可能多有不同，我個人習慣上撰寫區域會放在右邊，工作目錄的樹狀圖會放在左邊，但目前空空如也：
![hello world](hello-world-3.png#center)

　

接著點擊左上角的 `≡` 叫出系統選單，選擇 「終端機」 → 「新增終端」 ，會在下方的區域開啟一個終端機視窗，如果是 Windows ，可能就會預設以 PowerShell 開啟，另外此時的工作目錄 (紅色箭頭處) 就是資料夾的位置：
![hello world](hello-world-4.png#center)

　

按下這個按鈕，可以在資料夾裡新增一個檔案：
![hello world](hello-world-5.png#center)

　

例如我們命名為 `test.py` ，這個 py 副檔名會跟 python 有所關聯，命名完成後，右邊的撰寫區會立刻**開啟**這個檔案：
![hello world](hello-world-6.png#center)

　

我們可以在這個檔案裏面寫上 `print('Hello world!')` ，並且按下 `ctrl+s` 存檔：
![hello world](hello-world-7.png#center)

　

接著在下方終端機視窗中輸入 `python test.py` 來**執行**這個檔案：
![hello world](hello-world-8.png#center)
>嚴格來說，應該這樣下指令 `C:\python312\python.exe C:\Users\USER9\Desktop\demo\test.py` ，也就是 `python.exe 的位置，空一格， py 檔的位置`。
>
>　
>
>但是我們有把 python 加入環境變數，可以簡寫成 `python` 或甚至是 `py` ，而後面的 py 檔的位置，則可以直接用**相對路徑**執行檔案。

　

搭啦！下一行出現的文字就是程式執行的結果了！恭喜已經寫好最基礎的程式了！
![hello world](hello-world-9.png#center)
>這邊**開啟**指的是把檔案以編輯器打開，並撰寫或修改內容；而**執行**則表示，在終端機視窗中以 `python + 檔案` 的命令顯示出該程式的結果。要留意一下差別。

***
## 關於 Anaconda
{{< figure src="https://prod-backend-company-uploads-transcend-io.s3.amazonaws.com/8d6dc27b-6eef-4afc-8e75-1e1ac922e35f/e8d51866-cab8-4ea9-9ab7-b72dea449a4f" width="20%" alt="Anaconda-logo" align="center" >}}
約莫在 2020 年前後，所有的 Python 入門教學文不約而同的都請初學者使用 Anaconda 這套組合，最近可能熱度下降了，這個組合會幫忙安裝 python 以及各種關於資料科學等等的套件都會預先安裝，另外也會安裝他自有或是 pycharm 、瀏覽器版本的 Jupyter 等編輯器。另外有別於原生版的 pip ， Anaconda 安裝套件的方法改為自有的 `conda 套件名稱` ，應該是可以基於 Anaconda 自身的套件上在加以安裝或修改的版本。

所以等於安裝一個 Anaconda ，就可以少設定 Python 的安裝位置，不用自行安裝 pip 套件，不用自行安裝 VS code 。如意算盤打好之後發現如果要做點更深入的開發，例如安裝 Tensorflow 、建立虛擬環境，或是更新版本都會碰到不少的問題；更甚至在 Anacoda 之下又安裝了原生版的 Python ，有可能導致兩個都無法使用。

不過如果是習慣 R 的使用者，因為介面聽說和 R 都很像，我個人是不知道，但或許是個很好上手的組合包也說不定。

![Anaconda-demo](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LOyzpLEM7O9qHDKoruqoXQ.png)

***
## 關於 Jupyter
![Jupyter-logo](https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Jupyter_logo.svg/207px-Jupyter_logo.svg.png#center)
這也是一個大家都推薦新手的安裝的東西，它本身是一個 python 的套件，並不是集成式組合，也就說要先完成 pytohn 的安裝後，接著在 cmd 或 powershell 下指令 `pip install notebook` 會開始安裝以 ipython 為名的相關套件，安裝完畢之後鍵入 `jupyter notebook` 指令就會直接以瀏覽器開啟介面：
{{< figure src="jupyter-demo-1.png" width="60%" alt="Jupyter-Notebook-Demo" align="center" >}}

把程式輸入在 [] 後面的表單欄位中，按下 `ctrl+Enter` 就可以單行執行，某些寫法中可以省略 `print()` 直接在欄位中寫入變數名，就可以直接顯示變數內容。

如果是 Pandas 的 DataFrame 會直接用表格顯示內容，一目了然：
![Jupyter-Notebook-Demo](jupyter-demo-2.png#center)

　

另外行跟行之間也是具有因果關係的，例如我們在第 13 行輸入 import pandas as pd 之前，就必須安裝 Pandas 的套件，如同第 11 行使用 `!pip install pandas` ，前面會有一個驚嘆號是 Jupyter 介面的特有寫法：
{{< figure src="jupyter-demo-3.png" width="70%" alt="Jupyter-Notebook-Demo" align="center" >}}

　

總之這個介面適合給，看到一片黑、只能輸入指令又沒有滑鼠視窗，會覺得腦中一片空白的使用者使用。不過這個程式寫出來的附檔名是 `ipynb` ，如果要直接執行，必須在 jupyter 介面匯出成 `py` 檔，不然預設執行的程式應該是瀏覽器。

***
## 關於 Google Colab
Colab 是 Google 提供的一個線上版的 Jupyter ，預設的儲存位置是自己的 Google 雲端硬碟。使用這個好處是所有環境都會建立在 Google 提供的環境中
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
***
## 關於 Google Colab
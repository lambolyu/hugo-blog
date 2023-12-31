+++
title = '這裡是哪裡'
slug = '2023-09-new-blog-by-hugo'
date =  2023-09-03T11:41:07+08:00
draft = false
isCJKLanguage = true
showToc = true
TocOpen = true
categories = ['Web']
tags = ['Hugo','Netlify','Blog','PaperMod','Github','部落格','靜態文章']
+++

回醫院工作也兩三年了，因為庫存管理的工作包含了大量重複性的數據操作與程式操作，陸陸續續也從 google 學寫了不少小程式解決工作，提高了超多效率。但是因為主管個性比較 **~~機~~** 積極，我並不想把這些程式讓很多人知道，低調自用就好。
***
## 找個地方記錄
雖然是低調自用，不過還是想記錄一下這幾年寫過什麼，踩了多少坑，用什麼方法處理。回首自己的心路歷程，也有利於加速未來重構的效率。

「想要記錄」 大概是 2020 年左右的事，當時 Medium 當紅，在 2023 年的現在多數評價只剩下方便編寫的功能，不管是 SEO ，或者是付費政策都令當初的作家們滿不開心的。

於是決定使用 Static Side Generation 來架構 Blog ，雖然我比較熟悉的後端語法，但是如果可以因此學到比較新穎的前端技術也算有所成長。
***
## Hugo 和 Zola 要選誰
因為從來沒碰過前端的程式，不管是選了哪個都讓我摸不著頭緒好久。經過兩三天的自行摸索後，有了以下的心得：
>Hugo
1. 使用者族群眾多，themes 也很多
2. 產生的架構看起來龐大，資料夾看起來很多很可怕
3. 模板語言似乎以 GO 撰寫，我比較沒有經驗
>Zola
1. 使用者族群超少，themes 又少又醜
2. 產生的結構乾淨，資料夾看起來比較少
3. 模板語言是 tera ，看起來跟 django 類似，我比較有經驗

因為是初學的關係，儘管對 Zola 的初印象比較好，後來還是選擇使用 Hugo ，至少踩坑了比較多得社區資源和使用經驗可以查詢。
***
## Hello Hugo!
決定好之後就照著[官方文件](https://gohugo.io/)的指引一步一步來。

我的開發平台是 Windows ，先安裝了 [Scoop](https://scoop.sh/) ，接著在 cmd 視窗中輸入：
```powershell
scoop install hugo-extended
```

按下 `shift` + 滑鼠右鍵，選擇`在這裡開啟 PowerShell 視窗`，利用以下指令建立資料夾：
```bash
hugo new site blog
```

生成的資料夾中會包含一個 config 檔案 `hugo.toml`，是 toml 格式的設定檔。這個格式無論是在 Hugo 中或是在 Zola 中都一樣。

再來需要下載佈景主題 themes ，除了找到自己覺得好看的，也可以參考該佈景主題在 Github 中的星數，畢竟都選了熱門的 SSG ，當然也要選熱門的 themes ，一切都是為了讓新手如我有更多資源可以查詢。

我選的是 [Hugo-Papermod](https://github.com/adityatelange/hugo-PaperMod) ，按照官方指引在剛建立的 blog 資料夾底下先初始化 git ，再利用 submodule 把 themes 下載下來。
```bash
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
***
## hugo.toml
這時候的 `hugo.toml` 空空如也，下載回來的 themes 也還需要客製化之後才能繼續使用。

PaperMod 指引中預設的設定檔格式是 yml (yaml)，在 theme installation 指引中有提到的 `config.yml` ，其實就是 0.110.0 版本以後 Hugo 的 `hugo.toml` ，但後續的 Netlify 也使用 toml 作為設定檔，所以就統一使用 toml 格式不再做另外的轉換了。

依照目前的紀錄需求，不需要訪客分享文章、不需要訪客回應，甚至也不需要多國語言設定 (這樣要把文章拿去餵 ChatGPT 翻譯哈哈哈)，不需要個人介紹頁面。

PaperMod installation 最底下有個 [Sample config.yml](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#sample-configyml) ，可以自己利用 yaml 轉 toml 的轉檔程式，或是保留要的部分自己手動轉換 toml 也可以，以下是這個網站的設定檔：
```toml
baseURL = "https://lambo.tw/" #這個之後還會改，先暫定
languageCode = "zh-Hants-TW"
title = "Lambo’s Work Log"
theme = "PaperMod"
paginate = 15
enableInlineShortcodes = true
enableRobotsTXT = true
hasCJKLanguage = true
isCJKLanguage = true
enableEmoji = true
pygmentsUseClasses = true
defaultContentLanguage = 'zh-tw'

[minify]
disableXML = true

[outputs]
home = [ "HTML", "RSS", "JSON" ]

[permalinks]
posts = "/:year/:month/:slug/"

[params]
env = "production"
description = "紀錄使用python和php等多種程式處理醫院藥庫的日常工作情形。"
keyword = [
  "python",
  "php",
  "hospital",
  "pharmacy",
  "stock",
  "inventory",
  "pharmacist"
]
author = "lambo"
defaultTheme = "auto"
dateFormat = "2006/1/2"
ShowReadingTime = true
ShowWordCount = true
ShowPostNavLinks = true
ShowBreadCrumbs = true
ShowCodeCopyButtons = true

[params.homeInfoParams]
Title = "Lambo's Work Log"
Content = """
某醫院藥庫日常工作的程式紀錄<br>
很多的 Python 加上一點點麵條式 PHP<br>
完全不管程式效率和邏輯，可以用就好的自學者 (？)
"""

[[params.socialIcons]]
name = "facebook"
url = "https://www.facebook.com/lambo.lyu/"

[[params.socialIcons]]
name = "instagram"
url = "https://www.instagram.com/lambolyu/"

[params.assets]
disableHLJS = true

[markup.goldmark.renderer]
unsafe = true

[markup.highlight]
noClasses = false

[[menu.main]]
identifier = "archives"
name = "文章列表"
url = "/archives/"
weight = 10

[[menu.main]]
identifier = "tags"
name = "標籤"
url = "/tags/"
weight = 30

[[menu.main]]
identifier = "categories"
name = "分類"
url = "/categories/"
weight = 40

[[menu.main]]
identifier = "home"
name = "首頁"
url = "/"
weight = 0
```
***
## archives 和 search
可以留意到剛剛設定 `hugo.toml` 最底下的 menu 設定，就是 PeperMod 右上角的選單。
![PaperMod Menu](papermod-menu.png#center)
在 PaperMod 的 Features 指引中提到，需要自行新增 archives 和 search 的頁面，不過因為我暫時不需要搜尋功能，僅需設定 archives 就好：
```bash
hugo new content archives.md
```
新增一個 md 檔的頁面到 content 資料夾底下，並且修改文件內容：
```markdown
+++
title = "文章列表"
layout = "archives"
url = "/archives/"
summary = "archives"
+++
```
***
## categories 和 tags
有別於 archives 和 search ， Menu 中剩下的 categories 和 tags 屬於 Hugo 的原生功能，如果用剛剛的方式 new content ，會被 Hugo 當成是一篇文章。

所以如果要中文化或是客製，就要修改 layouts 中的模板了。

要建立客製的模板，可以參考 `\themes\PaperMod\` 中的相對位置，並複製一份到相對應的根目錄， Hugo 在渲染 (render) 網頁時會優先以根目錄的模板當作標準。如果直接改 themes 中的模板，將來 themes 更新時會被覆蓋掉。

舉例來說，現在要修改的檔案位在

`\themes\PaperMod\layouts\_default\term.html`

在對應的根目錄中，就要新增資料夾重新命名並複製一份檔案過去

`\layouts\` → `\layouts\_default\term.html`

接著我們要修改這份 html 檔案中的：

```html
<h1>{{ .Title }}</h1>
```

利用模板語言告訴 Hugo ，當變數 Title 為 Tags 時，標題應該改為標籤：

```html
{{ if eq .Title "Tags" }}
    <h1>標籤</h1>
{{- end }}
{{ if eq .Title "Categories" }}
    <h1>分類</h1>
{{- end }}
<!--<h1>{{ .Title }}</h1>-->
```

另外網頁最底下的版權說明雖然很重要，但其實並不好看，所以我也複製了一份 `\layouts\partials\footer.html` 進行 footer 的修改。


>這裡其實可以在 `\content\` 底下新增一個 `\content\tags\_index.md` 和 `\content\categories\_index.md` 然後分別在各自的 md 檔增加下面的內容
>```markdown
>+++
>title: "標籤"
>+++
>```
>只是我覺得 `\content\` 底下資料夾太多看起來不整潔，所以比較喜歡修改 layouts 。

## 開始用 Markdown 來寫文章吧！
新增文章的方法是在網站的根目錄底下執行：
```bash
hugo new content posts\文章標題.md
```
這個指令會在 `\content\posts\` 新增一份 Makdown 檔案，代表一篇文章。

但是 Makdown 檔案的前幾行會預設有一個被 `+++` 圍起來的段落：
```markdown
+++
title = '文章標題'
date = 2023-09-01T11:41:07+08:00
draft = true
+++
```
這個段落被稱為 Front Matters ，是用來讓 Hugo 解析文章時的設定。執行 `hugo new content` 的時候，會以 `\archetypes\default.md` 為模板生成一份 Makdown 檔案，所以直接修改 `default.md` 的 Front Matters 會讓之後省事不少。

因此我改了 `default.md` 的設定，其中有一項 slug 則是對應之前的 hugo.toml 的 permalinks 區塊，意思是這篇文章將在網址上面顯示 `/年/月/Front Matters設定的slug文字` ，這樣的設定有助於 SEO 排名。
```markdown
+++
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
slug = '{{ replace .File.ContentBaseName "-" " " | title }}'
date = {{ .Date }}
draft = true
categories = []
tags = []
+++
```
內文的編寫在 Front Matters 第二個 `+++` 的下一行就可以開始撰寫，寫文章的時候主要使用 Markdown語法，但如果覺得語法擴充性不足，其實還支援 html 原生語法，或是可以利用 [shortcodes](https://gohugo.io/content-management/shortcodes/) 來觸發 Hugo 內建的便利語法。例如內嵌 Instagram 、內嵌 Youtube 、甚至因為 Markdown 原生的圖片與法 `!()[]` 無法設定圖片大小及對齊方式，便可以利用 shortcodes 來改寫。
***
## 預覽測試
其實一邊寫，一邊可以用以下的指令在本地端建立一個網頁伺服器同步測試：
```bash
hugo server -D
```
`-D` 可以讓文章 Front Matters 是 `draft = true` 也就是草稿狀態的文章一同被渲染成靜態網頁，預設是草稿不會出現。

指令的結果，最末幾行的文字會如下：
```bash
(前略)
...
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
網址 `http://localhost:1313/` 就是 Hugo 建立的臨時伺服器，可以直接使用瀏覽器瀏覽建立出的網頁。而且滿神奇的，只要根目錄裡面的檔案有所更動，不必關閉再重新執行，就可以再次呈現新的結果，十分方便。甚至也可開著網頁伺服器來邊寫 Markdown 文章，邊預覽文章效果。
***
## 部署 (deploy) 網頁

完成第一篇文章後，終於可以把網站上線了（不過這個步驟其實沒有文章可以進行）。

網站的上線有兩種方式：

>註冊 Github 後，新增一個 repository 成為 Github Pages ，此時的 repository 名稱必須是 `github名稱.github.io` 。接著在網頁的根目錄執行指令 `hugo` ，就會在根目錄底下產生一個 `\public\` 的資料夾，然後把這個資料夾裡面的所有網頁直接 push 上 Github Pages 就完成了。

但我比較喜歡下面的方法，不用每次部署都還要本地端跑一下 `hugo` ， `\public\` 底下 git remote 和 git push ， git clone 卻又抓不到原始的根目錄，無法異地作業。

>註冊 Github 後，新增一個 repository ，直接把根目錄上的所有內容 push 上去。接著利用 Netlity 新增一個 site 串聯 Github 的 repository 。如此一來只要每次的 git push ，就會觸發 Netlity 直接更新網站。

{{< figure src="deployed-by-netlify.png" width="80%" alt="deployed-by-netlify" align="center" >}}


將網站 git push 上去之前，必須跟 Netlity 說明我們用的 SSG 是 Hugo ，因此根據 [官方指引](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/) ，我們需要在根目錄底下新增 `netlify.toml` ，用來向 Netlity 溝通 Hugo 的環境，這個檔案會隨著 Hugo 的版本有所差異。

```ini
[build]
publish = "public"
command = "hugo --gc --minify"

[build.environment]
HUGO_VERSION = "0.118.2"

[context.production.environment]
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"

[context.split1]
command = "hugo --gc --minify --enableGitInfo"

[context.split1.environment]
HUGO_ENV = "production"

[context.deploy-preview]
command = "hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL"

[context.branch-deploy]
command = "hugo --gc --minify -b $DEPLOY_PRIME_URL"

[context.next.environment]
HUGO_ENABLEGITINFO = "true"

[[redirects]]
from = "/npmjs/*"
to = "/npmjs/"
status = 200
```
準備完成之後就可以將整個根目錄 push 上 Github repository ，並且在 Netlity 設定連結開始部署網站。
部署成功之後， Netlity 會接著設定一個自定義的短網址： 自訂名稱.netlify.app ，如果短期之內沒有要買網域，要記得把剛剛的網址複製起來，然後修改網站根目錄的 hugo.toml，否則網站連結點下去會對應不到頁面。
```ini
baseURL = "https://自訂名稱.netlify.app/"
```
整個網站的建立差不多就是這樣了，剩下的就是開始累積文章量，日後在逐漸增加網站的花邊功能，例如：訪客回應，搜尋全站內容，加入 google analytics 等等。
## 網站誕生啦！

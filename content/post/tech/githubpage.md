+++
author = "Rico"
title = "How to use gitbook set up github pages"
date = "2019-07-06"
description = "How to set up github pages step by step"
tags = [
    "github",
    "gitbook",
    "markdown",
]
categories = [
    "tech",
]
archives = "2019-07"
+++

## 先情提要

在寫這篇的時候我使用的是gitbook，但礙於在使用gitbook [plugin-disqus](https://github.com/GitbookIO/plugin-disqus)的時候遇到一些[issue](https://github.com/GitbookIO/plugin-disqus/issues/10)

簡單講就是gitbook已經商業化，原本的[開源專案](https://github.com/GitbookIO/plugin-disqus/issues/10)已經3年沒有更新了

gitbook-cli最多只有支援到`3.2.2`，但是plugin-disqus需要`4.0.0`以上的版本

最主要還是沒有人維護的話，自己使用是蠻痛苦的一件事，所以就搬家到Hugo了。

但這篇gitbook安裝使用的文章還是保存起來做紀念。

## 正文開始


我想許許多多的工程師都會希望自己寫的文章可以有人拜讀，即使自己還不夠成熟

寫一手漂亮的文章必須要有對應的工具輔助

我最為推薦的是gitbook ，不過gitbook大部分的轉為線上的商業版本

於是這裡做個小小的教學如何使用gitbook產生網頁並且在github pages上瀏覽

當然不能忘記分享原本是在哪裡學的 [Publish a book using Gitbook](https://www.youtube.com/watch?v=ooXRd8RmuXY&list=LLaRPD_Md48Koxo2cE3nBSmA)

## gitbook editor

gitbook editor 是一個非常簡單好用的編輯文字的程式，不過要注意的一點是他不能把markdown的文章格式轉化爲html的格式\(或許藏在我沒注意到的地方\)

之後我會再講解怎麼做轉換

download gitbook editor [download](https://legacy.gitbook.com/editor)

因為我們沒有要用商業線上版 所以選擇Do that later

![gitbook-login.png](/gitbook-login.png)

創一個新的book
![createbook.png](/createbook.png)

開啟terminal 預設的位置在`cd ~/GitBook/Library/Import/{newbook-name}`

安裝node 不過我們需要的是npm

`brew install node`

`npm -v`

確認npm安裝完成後開始安裝gitbook

`sudo npm install -g gitbook-cli`

接下來就可以開始用gitbook指令做預覽 不過這個預覽其實看到的跟gitbook editor上內容一模一樣

不過現在是在瀏覽器上看到 基本上就是等等在github pages上看到的內容一樣

`gitbook serve`

可以看到只要去localhost:4000 就可以看到內容了

![gitbook-serve.png](/gitbook-serve.png)

只要開啟瀏覽器去[http://localhost:4000](http://localhost:4000) 就可以看到你剛剛編輯過的內容了

![gitbook-preview.png](/gitbook-preview.png)

只要有下`gitbook serve`的指令都會創造出一個裝滿靜態檔案的目錄`_book`

當然如果沒有想要做任何預覽的話可以選擇下`gitbook build`的指令就可以直接產出`_book`

![bookhtml.png](/bookhtml.png)

去github創一個新的repo 複製url或者透過ssh key加到本地端的repo裡面

`git remote add origin {repo-url}`

`git push -u origin master`

![gitpush.png](/gitpush.png)

之後你會發現推上去的repo裡面並沒有`_book`的目錄

`cat .gitignore` 後會發現git把`_book`都忽略了

不過不用太擔心 去github repo的setting，當然repo本身要要設定為public

往下看可以看到github pages，點開後可以發現可以把靜態檔案放在`/docs`的目錄裡

不過只能在master branch而已

現在不能選是正常的

![gitdocs.png](/gitdocs.png)

所以要回terminal下以下指令

`mv _book docs`

`git add .`

`git commit -m "add docs"`

`git push origin master`

之後再回到github setting那邊可以發現master branch /docs folder這個選項可以選擇了

點完save後就會給你一個url link，他必須要等一段時間 等到變成綠色就代表完成了

之後就可以進去看網頁的成品了




## Vim
又或者可以選擇使用家喻戶曉的文字編輯器Vim，而且我在使用gitbook editor的時候發現會有閃跳的情況

先創一個新的目錄並且進入該目錄

`mkdir ~/newBook && cd newBook`

起始gitbook，他會自動create README.md and SUMMARY.md這兩個file

`gitbook init`

當然也不能忘記啟用git

`git init`

`git add remote origin {repo-url}`

打開SUMMARY.md 可以發現gitbook特有的目錄分類方式，從下面範例可以發現`Introduction`這個文章來自root目錄的README.md 檔案

```
# Summary

* [Introduction](README.md)
```

以我這個gitbook本身為例，如果要寫得一模一樣要如下

```
# Summary

* [Introduction](README.md)
* [Tech-Sharing](tech-sharing/README.md)
    * [how to set up github pages](tech-sharing/githubpage.md)
```

**插入圖片** 先創一個目錄專門放圖片

`mkdir assets && cd assets`

之後存取圖片並且命名為`shutUp.png`，並且在`.md`的文章裡面輸入

`![shutUp.png](/shutUp.png)`

之後再透過`gitbook serve`就可以看到圖片了

**插入連結** 其實只差一個驚嘆號而已，如下範例 顯示出來為Google並且為藍色文字

`[Google](https://google.com)`

範例 [Google](https://google.com)



之後如何把編輯好的book上到github設定都一樣

個人認為gitbook editor vs Vim最大的差異為放圖片這件事

因為Vim放圖沒辦法做拖曳這麼直覺的事情 但gitbook editor在我用的mac下遇到大圖會跳段落 看起來像是bug

而隔壁棚的vscode可以做到剛截圖就可以直接插入到markdown的文章裡，[傳送門](https://github.com/telesoho/vscode-markdown-paste-image)

Vim的 隨然還沒實際用過 不過應該可以省不少時間[傳送門](https://github.com/ferrine/md-img-paste.vim)

## 將 Wikidown 推上 heroku 成為雲端系統

如果您已經有個專案，並且將檔案都加入 git  裡面了，就像下列文章中所做的事情一樣，那麼恭喜您，您已經具備了將檔案上傳到 heroku 上的大部分基礎，只要在補上幾個動作就行了。

* [將 Wikidown 推上 github 成為開源靜態網站](../pmag201503/focus3.html)

### 申請帳號並安裝軟體

要將您的 node.js 系統「安裝」到 heroku 平台上，必須依靠 git 工具，還有一個 heroku 所出版的軟體，稱為 Heroku Toolbelt for Windows ，您可以到下列網址中下載這個軟體。

* <https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up>

安裝好之後，您必須上 heroku 網站 <https://www.heroku.com/> 上申請帳號密碼，然後用 `heroku create <project>` 這個指令在 heroku 上創建一個專案，接著用 `git push heroku master` 將您的 master 版本推到 heroku 上，然後用 `heroku open` 指令打開該專案的網址，就可以看到您的專案已經在 heroku 上面運行了。

```
heroku create wikidown
git push heroku master
heroku open
```

當然、這是在您的專案與程式完全正確的情況之下才行的，不過、到底要上傳到 heroku 上的專案要如何才算是正確的呢？除了您可以用 node.js 將該專案跑起來之外，還必須加上一些設定。

第一個設定是您必須在專案的根目錄中加入一個名為 Procfile 的檔案，這個檔案用來告訴 heroku 執行您專案的指令。

檔案：Procfile

```
web: node wikiServer.js
```

還有您必須要寫好 node.js 需要的 package.json 檔案，以下是 wikidown 的 package.json 檔案之內容，這樣 heroku 才知道要為你準備好哪些版本的套件。

檔案： package.json

```javascript
{
  "name": "wikidown",
  "version": "1.0.0",
  "description": "Wikidown = wiki+markdown",
  "main": "wikiServer.js",
  "dependencies": {
    "serve-index": "~1.6.0",
    "body-parser": "^1.10.1",
    "cookie-parser": "1.3.3",
    "express-session":"1.10.1",
    "express": "4.11.0"
  },
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node wikiServer.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/ccckmit/markdown1.git"
  },
  "keywords": [
    "wiki",
    "markdown",
    "blog"
  ],
  "author": "ccckmit",
  "bugs": {
    "url": "https://github.com/ccckmit/markdown1/issues"
  },
  "homepage": "https://github.com/ccckmit/wikidown1"
}
```

還有很重要的一點是，您的 node.js 伺服端程式，必須使用 process.env.PORT 這個環境變數來開 port ，否則將會因為開錯 port 而無法運作。以下是 wikidown 專案打開 port 部份的程式碼：

```javascript
var port = process.env.PORT || 3000; // process.env.PORT for Heroku
app.listen(port);
```

如果您想在上傳之前先驗證您的系統是否能在 heroku 上正常運行，可以用 `foreman start` 這個指令進行測試，該指令會模擬 heroku 的環境來執行您的專案。執行完該指令後請開啟 `http://localhost:5000` 這個 網址來檢視您的系統。

如果一切沒問題，就可以按照本文一開頭的指示，用下列指令將系統上傳到 heroku 上運行。

```
heroku create wikidown
git push heroku master
heroku open
```

假如還是有問題的話，建議您參考 heroku 網站上針對 node.js 寫的 [getting-started-with-nodejs](https://devcenter.heroku.com/articles/getting-started-with-nodejs) 這篇文章，並且一步一步的照做一遍，應該就會瞭解如何將專案放上 heroku 平台的方法了。

### 後記

以下是筆者將 wikidown.js 系統上傳到 heroku 上的教學錄影，對於想發布專案到 heroku 上的人而言，應該會很有參考價值。

* [YouTube影片：如何將 node.js 的網站發布到 heroku 雲貒平台上 -- 以 wikidown.js 的發布為例](http://youtu.be/JpkTt9qAvXA)


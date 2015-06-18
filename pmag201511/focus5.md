## koa -- 建構 web 網站的核心框架

### 簡介

* 說明：  express 原班人馬創造支援 ES6 新語法 yield 的套件

* 官網專案： <https://github.com/koajs/koa>

* 推薦文章：[A Simple CRUD Demo with Koa.js
Sunday, January 12, 2014](http://weblogs.asp.net/shijuvarghese/a-simple-crud-demo-with-koa-js?hc_location=ufi)

* 推薦影片： James Moore (共十集） -- <http://knowthen.com/category/node-js/>

* 官方範例： <https://github.com/koajs/examples>

* single page app : <https://medium.com/@adam_bickford/creating-a-basic-site-with-koa-pt-1-f3e1711f7a9>


### Koa 與 Express 相關套件的對應表

| \ | Koa 相關套件 | Express 相關套件 |
|-----|------|------------|
| 中間件 | koa | connect |
| 路徑比對 | koa-router | express |
| 檔案靜態化 | koa-static, koa-static-folder | express.static |
| 資料夾索引 | koa-serve-index | serve-index |
| 檔案輸出入 | co-fs, mz/fs, save-to | fs |
| post body 處理 | koa-bodyparser, co-busboy | body-parser |
| session 管理 | koa-session | express-session |


### 官方範例

請先安裝 git, io.js 並用下列指令下載 koa.js 官方範例後，再開始執行後面給的例子。

```
$ git clone https://github.com/koajs/examples.git
$ cd examples
$ ren examples koa-ex
$ npm install
$ npm install swig
```

### 範例: Hello World

* 原始碼： <https://github.com/koajs/examples/tree/master/hello-world>
* 使用方法：執行 iojs app 之後看 http://localhost:3000/

### 範例: 

* 原始碼： <https://github.com/koajs/examples/tree/master/stream-file>
* 使用方法：執行 iojs app 之後看 http://localhost:3000/README.md

## 範例: templates

* 原始碼： <https://github.com/koajs/examples/tree/master/templates>
* 使用方法： 執行 iojs index 之後看 http://localhost:4000/

### 範例: 網誌

* 原始碼： <https://github.com/koajs/examples/tree/master/blog>

```
D:\git\koa-ex>npm install swig
swig@1.4.2 node_modules\swig
├── optimist@0.6.1 (wordwrap@0.0.2, minimist@0.0.10)
└── uglify-js@2.4.19 (uglify-to-browserify@1.0.2, async@0.2.10, yargs@3.5.4,
source-map@0.1.34)

D:\git\koa-ex>cd blog

D:\git\koa-ex\blog>iojs index
listening on port 3000
  <-- GET /post
  --> GET /post 404 23ms -
  <-- GET /
  --> GET / 200 116ms -
  <-- GET /post/new
  --> GET /post/new 200 18ms -
  <-- POST /post
  --> POST /post 302 47ms -
  <-- GET /
  --> GET / 200 10ms -
  <-- GET /post/new
  --> GET /post/new 200 40ms -
  <-- POST /post
  --> POST /post 302 32ms -
  <-- GET /
  --> GET / 200 25ms -
  <-- GET /post/0
  --> GET /post/0 200 8ms -
  <-- GET /post/1
  --> GET /post/1 200 7ms -
  <-- GET /post/new
  --> GET /post/new 200 5ms -

```

執行結果

![](koa_blog.jpg)



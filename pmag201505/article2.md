## 簡單留言系統 -- 使用單頁技術

### 簡介

最近在 HTML5 的加持與前端框架越來越強大的影響下，單頁技術開始快速興起，但是我們卻很少看到單頁技術的完整範例。

為了在上課時示範給學生看，筆者寫了一個非常簡單的單頁留言系統。

為了簡單起見，該系統沒有用到資料庫，暫時將留言都存在記憶體的 JSON 物件裡面。

這個專案已經上傳到 github 上，以下是專案的網址。

* 專案: <https://github.com/ccckmit/KoaSpaNote/>

我們在後端使用了 Koa 這個採用 yield 語法的框架，然後在前端採用 bootstrap 和 jQuery 這兩個框架，並且用 AJAX 作為前後端的溝通方法。

這個留言系統可以新增、修改、或列出留言，但我們沒有實做刪除功能，以下是列出留言的畫面。

![圖、留言列表](noteSpaList.jpg)

然後我們可以按某一筆留言進去編輯，以下是編輯時的畫面。

![圖、編輯第二筆留言](noteSpaEdit.jpg)

我們也可以按『新增記事』按鈕去增加新的留言，如下所示。

![圖、新增留言](noteSpaNew.jpg)

新增完之後又會自動跳回列出留言的畫面如下。

![圖、新增後留言列表](noteSpaList2.jpg)

在以上的互動過程中，伺服器會印出下列訊息。

```
D:\git\KoaSpaNote>iojs KoaNoteServer.js
noteServer started at port : 3000
get /web/note.html
get /web/main.css
/list path=/list
/list req=[object Object]
/list notes=[{"title":"title1","body":"body1"},{"title":"title2","body":"body2"}
,{"title":"title3","body":"body3"}]
response: code=200
[{"title":"title1","body":"body1"},{"title":"title2","body":"body2"},{"title":"t
itle3","body":"body3"}]

get /web/favicon.ico

  Error: ENOENT: no such file or directory, open 'D:\git\KoaSpaNote\web\favicon.
ico'
      at Error (native)

response: code=200
{"title":"title1","body":"body1"}

response: code=200
{"title":"title2","body":"body2"}

response: code=200
{"title":"title2","body":"body2\n\nxxx"}

post new title 1:new body 1
response: code=200
write success!

/list path=/list
/list req=[object Object]
/list notes=[{"title":"title1","body":"body1"},{"title":"title2","body":"body2\n
\nxxx"},{"title":"title3","body":"body3"},{"title":"new title 1","body":"new bod
y 1"}]
response: code=200
[{"title":"title1","body":"body1"},{"title":"title2","body":"body2\n\nxxx"},{"ti
tle":"title3","body":"body3"},{"title":"new title 1","body":"new body 1"}]

```

這就是整個留言系統的功能了，接著讓我們來看看前後端的程式碼。

### 前端網頁

檔案： note.html

```html
<html>
<head>
  <meta charset="utf-8" />
  <link href="favicon.ico" rel="icon">
  <link href="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css" rel="stylesheet">
  <link href="main.css" rel="stylesheet">
  <title>簡易記事本</title>
</head>
<body onload="init()">
  <nav class="navbar navbar-inverse navbar-fixed-top">
    <div class="container">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <div class="navbar-header" style="color:#C0C0C0">
        簡易記事本
      </div>
      <div id="navbar" class="navbar-collapse collapse">
        <form class="navbar-form navbar-right">
          <div class="form-group">
            <input id="filepath" type="hidden" class="form-control" placeholder="filepath" aria-describedby="basic-addon1">
            <button class="btn btn-success" type="button" onclick="list()">所有記事</button>
            <button class="btn btn-success" type="button" onclick="newNote()">新增記事</button>
          </div>
        </form>     
      </div>
    </div>
  </nav>

  <div id="panelList" class="tab-pane panel">
  </div>
  <div id="panelEdit" class="tab-pane panel">
    標題： <input id="editTitle" type="text" value="">
    <br/>
    <textarea id="editBody" class="form-control" style="width:100%; height:60%"></textarea>
    <button class="btn btn-success" onclick="save()">儲存</button>
  </div>
  <div id="panelNew" class="tab-pane panel">
    標題： <input id="newTitle" type="text" value="">
    <br/>
    <textarea id="newBody" class="form-control" style="width:100%; height:60%"></textarea>
    <button class="btn btn-success" onclick="add()">新增</button>
  </div>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
  <script src="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"></script>
<script>
$('.panel').css( "display", "none");
Server = {
  timeout : 4000
};
Server.save=function(file, text) {
  $.ajax({
    type: "POST",
    url: "/note/"+file,
    timeout: this.timeout,
    data: { text: text }
  })
  .done(function(data) {
    alert( "存檔完成!");
  })
  .fail(function() {
    alert( "存檔失敗！" );
  });
}
Server.load=function(file) {
  return $.ajax({
    type: "GET",
    url: "/note/"+file,
    timeout: this.timeout,
    data: {}
  });
}
function init() {
  list();
}
function switchPanel(name) {
  $('.panel').css( "display", "none");
  $('#'+name).css( "display", "block");
}
function list() {
  switchPanel('panelList');
  $.ajax({
    type: "GET",
    url: "/list",
    timeout: this.timeout,
    data: {}
  })
  .done(function(data) {
    var notes = JSON.parse(data);
		$('#panelList').empty();
		$('#panelList').append("<ol>");
    for (var i in notes) {
      var title = notes[i].title;
      var body  = notes[i].body;
      $('#panelList').append('<li><a href="javascript:edit('+i+')">'+title+"</a></li>")
    }
		$('#panelList').append("</ol>");
  });
}
var noteID;
function edit(id) {
  switchPanel('panelEdit');
	noteID = id;
  $.ajax({
    type: "GET",
    url: "/note/"+id,
    timeout: this.timeout,
    data: {}
  })
  .done(function(data) {
    var note = JSON.parse(data);
    $('#editTitle').val(note.title);
    $('#editBody').val(note.body);
  });
}
function save() {
  var title = $('#editTitle').val();
  var body  = $('#editBody').val();
  $.ajax({
    type: "POST",
    url: "/note/"+noteID,
    timeout: this.timeout,
    data: {
      title:title,
			body:body
    }
  })
  .done(function(data) {
	  alert('存檔完成!');
	});
}
function newNote() {
  switchPanel('panelNew');
  $('#editTitle').val('');
  $('#editBody').val('');
}
function add() {
  var title = $('#newTitle').val();
  var body  = $('#newBody').val();
  $.ajax({
    type: "POST",
    url: "/new",
    timeout: this.timeout,
    data: {
      title:title,
			body:body
    }
  })
  .done(function(data) {
	  alert('新增完成!');
		list();
	});
}
</script>
</body>
</html>
```

### 後端伺服器

檔案： KoaNoteServer.js

```javascript
var http    = require('http');
var url     = require('url');
var koa     = require('koa');
var fs      = require('mz/fs');
var path    = require('path');
var bodyParser = require("koa-bodyparser"); // 參考：http://codeforgeek.com/2014/09/handle-get-post-request-express-4/
var route   = require('koa-route');
var c       = console;

var app = koa();
app.use(bodyParser());

var notes = [ {title:'title1', body:'body1'}, 
              {title:'title2', body:'body2'}, 
              {title:'title3', body:'body3'} ];

function response(res, code, msg) {
  res.status = code;
  res.set({'Content-Length':''+msg.length,'Content-Type':'text/plain'});
  res.body = msg;
  c.log("response: code="+code+"\n"+msg+"\n");
}

app.use(route.get('/', function*() {
  this.redirect('/web/note.html');
}));

var mime = { ".css":"text/css", ".html": "text/html", ".htm":"text/html", ".jpg":"image/jpg", ".png":"image/png", ".gif":"image/gif", ".pdf":"application/pdf"};

function fileMimeType(path) {
  for (var tail in mime) {
    if (path.endsWith(tail))
      return mime[tail];
  }
}

app.use(route.get(/\/web\/.*/, function *toStatic() {
  var req = this.request, res = this.response;
  c.log('get %s', this.path);
  var mimetype = fileMimeType(this.path)
  if (mimetype) this.type = mimetype+";";
  this.body = fs.createReadStream(__dirname+this.path);
}));

app.use(route.get("/list", function *list() {
  c.log("/list path=%s", this.path);
  c.log("/list req=%s", this.request);
  c.log("/list notes=%j", notes);
  response(this.response, 200, JSON.stringify(notes));
}));

app.use(route.get("/note/:id", function *edit(id) {
  var i = parseInt(id);
  response(this.response, 200, JSON.stringify(notes[i]));
}));

app.use(route.post("/note/:id", function *save(id) {
  var i = parseInt(id);
  var title = this.request.body.title;
  var body = this.request.body.body;
  notes[i] = { title:title, body:body };
  response(this.response, 200, JSON.stringify(notes[i]));
}));

app.use(route.post('/new', function *newNote() {
  var req = this.request, res = this.response;
  var title = this.request.body.title;
  var body = this.request.body.body;
  c.log('post %s:%s', title, body);
  notes.push({title:title, body:body});
  response(res, 200, 'write success!');
}));

http.createServer(app.callback()).listen(3000);

c.log("noteServer started at port : 3000");
```


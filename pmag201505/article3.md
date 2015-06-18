## 讓 wikidown 從 express 進化為 koa 版本

在 2015 年 3 月的時候，我們介紹了 wikidown.js 這個維基網誌系統，當時我們是採用 express 作為後端伺服器的。

但是後來筆者一邊學習 koa 套件與 yield 語法，決定一邊將 wikidown.js 改為 koa 版本，於是就將原本的 wikiServer.js 程式改用 KoaServer.js 替代。

以下是筆者的個人網站，該網站就是用 wikidown.js 架設的，如果您從以下的 http 版本進入，您將可以瀏覽，但不能編輯。 

* http 版入口 -- <http://ccc.nqu.edu.tw/>

但是筆者個人要編輯的時候，則必須從有安全加密的 https (ssl) 版本進入，然後輸入帳號密碼之後，就可以進行編輯與瀏覽了。

* https 版入口 -- <https://ccc.nqu.edu.tw/>

本系統的前後端程式碼，分別列出如下：

### 後端程式
檔案： KoaServer.js

```javascript
var http    = require('http');
var https   = require('https');
var c       = console;
var url     = require('url');
var fs      = require('mz/fs');
var koa     = require('koa');
var send    = require('koa-send');
var path    = require('path');
var bodyParser = require("koa-bodyparser"); // 參考：http://codeforgeek.com/2014/09/handle-get-post-request-express-4/
var session = require('koa-session');
var serveIndex = require('koa-serve-index');
var route   = require('koa-route');
var parse   = require('co-busboy');
var saveTo  = require('save-to');
var jslib   = require('./web/jslib');
var wdlib   = require('./wdlib');
var setting = require('./setting');
var passwords = setting.passwords;

var app = koa();
var webDir = path.join(__dirname, 'web');
var dbRoot = path.join(__dirname, 'db');

c.log("dbRoot=%s", dbRoot);

app.keys = ['xxxxxxxxxxxxxxxxx']; // 使用時請改掉
app.use(session(app));
app.use(bodyParser({formLimit:5*1000*1000, jsonLimit:5*1000*1000}));

function response(res, code, msg) {
  res.status = code;
  res.set({'Content-Length':''+msg.length,'Content-Type':'text/plain'});
  res.body = msg;
  c.log("response: code="+code+"\n"+msg+"\n");
}

function isPass(req) {
  c.log("loginToSave="+setting.loginToSave);
  if (setting.loginToSave === false) 
    return true;
  c.log('req.session.user='+req.session.user);
  return typeof(req.session.user)!=='undefined';
}

// 處理檔案上傳 : multipart upload
app.use(function *(next){
  var domain = this.request.url.split("/").pop();
  if ('POST' !== this.request.method) return yield next;
  if (!this.request.url.startsWith('/upload/')) return yield next;
  if (!this.request.header["content-type"].startsWith("multipart/form-data;")) return yield next;
  var part, parts = parse(this);
  var files = [], file;
  while (part = yield parts) {
    console.log('part=%j', part);
    if (typeof part.filename !== 'undefined') {
      files.push(file = path.join(dbRoot, domain, part.filename));
      console.log('uploading %s -> %s', part.filename, file);
      yield saveTo(part, file);
    }
  }
  this.body = files;
});

app.use(route.get('/', function*() {
  this.redirect('/web/wikidown.html');
}));

var mime = { ".html": "text/html", ".htm":"text/html", ".jpg":"image/jpg", ".png":"image/png", ".gif":"image/gif", ".pdf":"application/pdf"};

function fileMimeType(path) {
  for (var tail in mime) {
    if (path.endsWith(tail))
      return mime[tail];
  }
}

app.use(route.get(/.*/, function *toStatic() {
	if (!this.path.startsWith("/setting.js")) {
    var mimetype = fileMimeType(this.path)
    if (mimetype) this.type = mimetype+";";
    c.log('get %s', this.path);
    this.body = fs.createReadStream(__dirname+this.path);
	}
}));

app.use(route.post('/wd/:domain/:file', function*(domain, file) {
  var req = this.request, res = this.response;
  var obj = this.request.body.obj;
  if (!isPass(this)) {
    response(res, 401, 'Please login to save!')
    return;
  }
  c.log('post %s:%s', domain, file)
  // 寫入檔案 （通常是 *.wd 檔案）
  yield fs.mkdir(dbRoot+"/"+domain+"/").catch(function(){});
  yield fs.writeFile(dbRoot+"/"+domain+"/"+file, obj).then(function() {
    response(res, 200, 'write success!');
  }).catch(function() {
    response(res, 403, 'write fail!'); // 403: Forbidden
  });  // 將 *.wd 轉為 *.html 後寫入
  if (jslib.endsWith(file, '.wd')) {
    var html = wdlib.toHtml(obj, domain);
    var htmFile = file.replace(/\.wd$/, '.html');
    yield fs.writeFile(dbRoot+"/"+domain+"/"+htmFile, html);      
  }
}));

// 以下為 https 版本，必較不容易被竊取密碼，但是有 self signed 的認證問題
// 目前設定只有 https 版本可以登入並修改資料，http 版不行。
// var secureApp = app;

var secureApp = app;

secureApp.use(route.post("/login", function*() {
  var req = this.request, res = this.response;
  var p = this.request.body;
  if (req.protocol !== 'https') {
    response(res, 401, p.user+":login fail!");
    return;
  }  
  if (passwords[p.user] === p.password) {
    this.session.user = p.user;
    response(res, 200, p.user+":login success!");
  } else {
    response(res, 401, p.user+":login fail!");
  }
}));

secureApp.use(route.post("/logout", function*() {
  var req = this.request, res = this.response;
  this.session = null;
  response(res, 200, "logout success!");
}));

var port = process.env.PORT || 80; // process.env.PORT for Heroku

console.log('Server started: http://localhost:'+port);

http.createServer(app.callback()).listen(port);

var sslPort = 443;
https.createServer({
      key: fs.readFileSync('key.pem'),
      cert: fs.readFileSync('cert.pem'),
      // 以下只有 self signed 認證的方式需要
      requestCert: true, 
      ca: [ fs.readFileSync('csr.pem') ]
}, app.callback()).listen(sslPort);

console.log('Http Server started: http://localhost:'+sslPort);


```

### 前端程式

檔案： wikidown.js

```javascript
<html>
<head>
  <meta charset="utf-8" />
  <link href="favicon.ico" rel="icon">
  <link href="css/bootstrap.min.css" rel="stylesheet">
  <link href="css/highlight.default.min.css" rel="stylesheet">
  <link href="css/fileinput.css" media="all" rel="stylesheet"/>
  <link href="wikidown.css" rel="stylesheet">
  <title>Wikidown</title>
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
<!--        <a class="navbar-brand" href="#main:home">
          <span class="glyphicon glyphicon-home"></span> &nbsp;
        </a> -->
				<span id="titleHome" class="navbar-nav titleLink"><p><a href="#main:home" class="innerLink">首頁</a> /</p></span>
        <span id="titleHead" class="navbar-nav titleLink"></span>
      </div>
      <div id="navbar" class="navbar-collapse collapse">
        <ul class="nav navbar-nav navbar-right">
          <li class="dropdown">
            <a class="dropdown-toggle" data-toggle="dropdown"><span class="glyphicon glyphicon-globe"></span><!--<span class="caret">--></span></a>
            <ul class="dropdown-menu" role="menu">
              <li><a onclick="Mt.switchLang('us')">English</a></li>
              <li><a onclick="Mt.switchLang('tw')">中文</a></li>
            </ul>
          </li>
        </ul>
        <ul class="nav navbar-nav navbar-right hide" id="menuLogin">
          <li class="dropdown">
            <a class="dropdown-toggle" data-toggle="dropdown">登入</a>
            <ul class="dropdown-menu" role="menu">
              <li><a onclick="login()">登入</a></li>
              <li><a onclick="logout()">登出</a></li>
            </ul>
          </li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
          <li class="dropdown">
            <a class="dropdown-toggle" data-toggle="dropdown">分享</a>
            <ul class="dropdown-menu" role="menu">
              <li><a onclick="facebookShare()">Facebook</a></li>
              <li><a onclick="staticShare()">靜態連結</a></li>
            </ul>
          </li>
        </ul>
        <form class="navbar-form navbar-right hide" id="menuEdit">
          <div class="form-group">
            <input id="filepath" type="hidden" class="form-control" placeholder="filepath" aria-describedby="basic-addon1">
            <button class="btn btn-success" type="button" onclick="show()">預覽</button>
            <button class="btn btn-success" type="button" onclick="edit()">編輯</button>
            <button class="btn btn-success" type="button" onclick="save()">儲存</button>
            <button class="btn btn-success" type="button" onclick="upload()">上傳</button>
<!--        <button class="btn btn-success" type="button" onclick="share()">分享</button>-->
          </div>
        </form>     
      </div>
    </div>
  </nav>

  <div id="panelShow" class="tab-pane panel"> <!-- 預覽 -->
    <div id="htmlBox" style="overflow-y:scroll;height:90%;padding:10px;"></div>
  </div>
  <div id="panelEdit" class="tab-pane panel"> <!-- 編輯 -->
    <br/>
    <textarea id="wdBox" class="form-control" style="width:100%; height:80%"></textarea>
  </div>
  <div id="panelUpload" class="tab-pane panel" style="height:75%">
    <center>
    <div class="kv-main">
      <form enctype="multipart/form-data">
        <div class="form-group">
      <input id="imageUpload" name="imageUpload" type="file" multiple=true class="file-loading">
        </div>
      </form>
    </div>
    </center>
  </div>
  <div id="panelLogin" class="tab-pane panel"> <!-- 登入 -->
    <form class="form-signin" role="form">
      <input type="text" id="loginUser" class="form-control" required autofocus data-mt="User" placeholder="帳號"/>
      <input type="password" id="loginPassword" class="form-control" required data-mt="Password" placeholder="密碼"/>
      <button class="btn btn-lg btn-primary btn-block" type="button" data-mt="Login" onclick="Server.login()">登入</button>
    </form>
  </div>
  <script src="js/jquery.min.js"></script>
  <script src="js/bootstrap.min.js"></script>
<script>
$('.panel').css( "display", "none");

var domain, file, converter; // filepath, 
var wdNewFile = '# 標題：文件不存在\n\n您可以編輯後存檔！\n## 語法\n* [內部連結](innerLink.html)\n* [外部連結](link)';

function isLogin() {
  if (localStorage.wd_login !== "true") { // 注意：sessionStorage 不能跨頁面持續，所以得用 localStorage
    alert('未登入無法存檔上傳，請先登入!');
    login();
    return false;
  }
  return true;
}

Server = {
  timeout : 4000
};

Server.save=function(domain, file, doc) {
  $.ajax({
    type: "POST",
    url: "/wd/"+domain+"/"+file+".wd",
    timeout: this.timeout,
    data: { obj: doc },
		statusCode: {
      401: function() { // 401:Unauthorized
        localStorage.wd_login = "false";
        isLogin();
      }
    }
  })
  .done(function(data) {
    alert( "存檔完成!");
  })
  .fail(function() {
    alert( "存檔失敗！" );
  });
}

Server.load=function(domain, file) {
  return $.ajax({
    type: "GET",
    url: "../db/"+domain+"/"+file+".wd",
    timeout: this.timeout,
    data: {}
  });
}

Server.login=function() {
  $.ajax({
    type: "POST",
    url: "/login",
    timeout: this.timeout,
    data: { user:$('#loginUser').val(), password:$('#loginPassword').val() },
  })
  .done(function(data) {
    localStorage.wd_login = "true";
    alert( "登入成功!");
    $('#loginPassword').val('');
    edit();
  })
  .fail(function() {
    localStorage.wd_login = "false";
    alert( "登入失敗！" );
    login();
  });
}

Server.logout=function() {
  $.ajax({
    type: "POST",
    url: "/logout",
    timeout: this.timeout,
    data: {},
  })
  .done(function(data) {
    localStorage.wd_login = "false";
    alert( "登出成功!");
    show();
  })
  .fail(function() {
    alert( "登出失敗！" );
  });
}

window.onhashchange = function () {
  filepath = window.location.hash.substring(1);
  var parts  = filepath.split(':');
  domain = parts[0], file=parts[1];
  loadFile(domain, file);
  show();
}

window.onbeforeunload = function(){}

function md2html(md) {
  return converter.makeHtml(md);
}

function init() {
  converter = new Showdown.converter({ extensions: ['mathjax', 'table'] });
  if (window.location.hash === '')
    window.location.hash = '#main:home';
  window.onhashchange();
  if (window.location.protocol === 'https:') {
    // $('#navbar').css('display', 'none');
    $('#menuLogin').removeClass('hide');
    $('#menuEdit').removeClass('hide');
  }
  MathJax.Hub.Config({
      extensions: ["tex2jax.js"],
      jax: ["input/TeX", "output/HTML-CSS"],
			displayAlign: "left",
      tex2jax: {
        inlineMath: [ ['$','$'], ["\\(","\\)"] ],
        displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
        processEscapes: true
      },
      "HTML-CSS": { availableFonts: ["TeX"], scale: 130 }
    });
}

function switchPanel(name) {
  $('.panel').css( "display", "none");
  $('#'+name).css( "display", "block");
}

function templateApply(wd) {
  var templateNow = config.template[domain];
  if (templateNow===undefined)
    templateNow = config.template['default'];
  return templateNow.split('<%=wd%>').join(wd); // 不能用 replace(reg, str), 因為 str 中可能有參數 ＄...$ 符號，所以改用 split + join
}

function wdToHtml(wd, domain) {
  wd  = '\n'+wd+' ';
  wd  = wd.replace(/(\s)@\[\[([^\]]*?)\]\]\((.*?)\)/gi, '$1<a href="../db/'+domain+'/$3" class="innerLink">$2</a>'); // 內部檔案 [text](pathToFile.html)
  wd  = wd.replace(/(\s)@\[\[([^\]]*?)\]\]/gi, '$1<a href="../db/'+domain+'/$2" class="innerLink">$2</a>'); // 內部檔案 [pathToFile](pathToFile.html)
  wd  = wd.replace(/(\s)\!\[\[([^\]]*?)\]\]\((.*?)\)/gi, '$1<div class="figure"><img src="../db/'+domain+'/$3"/><p class="caption">$2</p></div>'); // 內部圖片 ![text](file)
  wd  = wd.replace(/(\s)\[\[([^\]]+?)\]\]\(([^:\)]+):([^:\)]+)\)/gi, '$1<a href="#$3:$4" class="innerLink">$2</a>'); // 內部連結 [text](../domain/file.html)
  wd  = wd.replace(/(\s)\[\[([^\]]+?)\]\]\((.*?)\)/gi, '$1<a href="#'+domain+':$3" class="innerLink">$2</a>'); // 內部連結 [text](file.html)
  wd  = wd.replace(/(\s)\[\[([^\]:]+):([^\]:]+)\]\]/gi, '$1<a href="#$2:$3" class="innerLink">$2:$3</a>'); // 內部連結 [[domain:file]]
  wd  = wd.replace(/(\s)\[\[([^\]:]+?)\]\]/gi, '$1<a href="#'+domain+':$2" class="innerLink">$2</a>'); // 內部連結 [file](file.html)
//  wd  = wd.replace(/(\s)\$([^$\n]+?)\$/gi, '$1\n<script type="math/tex">$2</'+'script>\n');// 數學式 $$[latex]$$,  刻意把 '</'+'script>' 分開，避免瀏覽器認為是真的 script 區塊
  return md2html(wd);
}

function login() {
  switchPanel('panelLogin');
}

function logout() {
  Server.logout();
}

function edit() {
  switchPanel('panelEdit');
}

function upload() {
  if (!isLogin()) return;
  switchPanel('panelUpload');
  $("#imageUpload").fileinput({
    uploadUrl: "/upload/"+domain,
    maxFileCount: 10,
    uploadAsync: false,
    uploadExtraData: {
        domain: domain,
        file: file
    }
  });
}

function absolute(base, relative) {
    var stack = base.split("/"),
        parts = relative.split("/");
    stack.pop(); // remove current file name (or empty string)
                 // (omit if "base" is the current folder without trailing slash)
    for (var i=0; i<parts.length; i++) {
        if (parts[i] == ".")
            continue;
        if (parts[i] == "..")
            stack.pop();
        else
            stack.push(parts[i]);
    }
    return stack.join("/");
}

function httpRef() {
  return window.location.href.replace('https:', 'http:');
}

function facebookShare() {
  window.open("https://www.facebook.com/sharer/sharer.php?u="+escape(absolute(httpRef(), '../db/'+domain+'/'+file+'.html'))+'&t='+document.title, '', 'menubar=no,toolbar=no,resizable=yes,scrollbars=yes,height=300,width=600')
}

function staticShare() {
//  window.open('../db/'+domain+'/'+file+'.html', '', 'menubar=no,toolbar=no,resizable=yes,scrollbars=yes,height=300,width=600')
  window.open(absolute(httpRef(), '../db/'+domain+'/'+file+'.html'), '', 'menubar=no,toolbar=no,resizable=yes,scrollbars=yes,height=300,width=600')
}

function show() {
  var wd = $('#wdBox').val();
  wd  = templateApply(wd);
  var html = wdToHtml(wd, domain);
  $('#htmlBox').html(html);
  $('pre code').each(function(i, block) {
    hljs.highlightBlock(block);
  });
  switchPanel('panelShow');
  if (typeof(MathJax) !== 'undefined') {
    MathJax.Hub.Queue(["Typeset",MathJax.Hub, "htmlBox"]);
	}
  showTitle();
}

function showTitleHead(domain) {
  var titleHead = config.title['default'];
  if (config.title[domain] !== undefined)
    titleHead = config.title[domain];
  var titleHtml = wdToHtml(titleHead, domain);
  $('#titleHead').html(titleHtml);
}

function showTitle() {
  if ($('h1').length > 0)
    $('title').html($('#titleHead').text()+' -- '+$('h1')[0].innerText);
  else if ($('h2').length>0)
    $('title').html($('#titleHead').text()+' -- '+$('h2')[0].innerText);
}

function loadFile(domain, file) {
  showTitleHead(domain);
  window.location.hash = '#'+domain+':'+file;
  Server.load(domain, file)
  .done(function(wd) {
    $('#wdBox').val(wd);
    show();
  })
  .fail(function() {
    $('#wdBox').val(wdNewFile);
    show();
  });
}

function save() {
  if (!isLogin()) return;
  var wd = $('#wdBox').val();
  Server.save(domain, file, wd);
}
/*
// 以下這種作法是為了加快速度，避免一開始載入 MathJax 花太久時間 ...
// 刻意把 '</'+'script>' 分開，避免瀏覽器認為是真的 script 區塊
$( document ).ready(function() {
    $('body').append('<'+'script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_SVG"></'+'script>');
});
*/
</script>

<script src="js/Showdown/Showdown.js"></script>
<script src="js/Showdown/extensions/table.min.js"></script>
<script src="js/Showdown/extensions/mathjax.js"></script>
<script src="js/highlight.min.js"></script>
<script src="js/fileinput.js" type="text/javascript"></script>
<script src="config.js"></script>
<script src="MathJax/MathJax.js?config=TeX-AMS-MML_SVG"></script>
</body>
</html>
```

### 結語

這個程式改成 koa 之後，所有的 callback 幾乎都不見了，而這也正是 koa 框架從 yield 語法上得到的優點阿！



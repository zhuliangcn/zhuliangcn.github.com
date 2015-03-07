---
layout:default
title:microblog的实现(node.js 0.10.34 + express 4.11.2 + mongodb 2.6.8)
---
## microblog的实现(node.js 0.10.34 + express 4.11.2 + mongodb 2.6.8) ##

《node.js开发指南》中的microblog实例是很多人学习node.js的开始，我也是，并且折腾了挺长时间。略有收获，但是感觉自己还需要在这条不归路上继续努力。现在把整个过程记录下来，也算阶段性总结，也希望有可能帮助一两个后来人。

《node.js开发指南》出版比较早（也感谢作者将其分享到网络上），因此其所有软件版本都比较早，也就造成了所谓的“折腾”。但是，所谓学习嘛，不折腾不成活...

这篇文章会一步一步详细记录折腾的过程，为了叙事方便，某些步骤可能与我当时执行的不完全一样，不过我**认为**可能不会是个问题...

### step1. 安装所需软件 ###

我用的开发环境如下：

- 计算机：组装机（CPU：i5-3470，RAM：8.00G）
- 操作系统：Windows 8.1专业版（中文）

软件我都是下载的我当前的最新版本：node.js v0.10.34和mongodb v2.6.8。 安装很简单，双击即可。装完是要设置环境变量的，过程应该都会。例如我装在了C:\Program Files中，环境变量里就添加：
<pre><code>C:\Program Files\nodejs\;C:\Program Files\MongoDB 2.6 Standard\bin
</code></pre>

例外一个值得一提的问题就是npm。我们知道，npm是node.js提供的方便的插件管理工具。但是我们出生在红旗下，生长在春风里，所以我们在用npm的时候会看到传说中的万里长城。爬长城很累人，好在淘宝在用node.js，而且做淘宝（平台）的人都是有着*业界良心*的人，于是他们提供了一个npm镜像，叫cnpm。请登陆：
<pre><code>https://npm.taobao.org/
</code></pre>
或者直接执行命令行执行命令：
<pre><code>$ npm install -g cnpm --registry=http://registry.npm.taobao.org
</code></pre>
后续执行`cnpm install`时可能会报错：`npm ERR! CERT_UNTRUSTED `。群里一个大拿——龙三——告诉我，需要把`https`源切换为`http`。可以执行如下两条命令：
<pre><code>$ cnpm config set registry http://registry.npm.taobao.org
$ cnpm conifg set disturl http://npm.taobao.org/dist
</code></pre>
然后建个目录，如`D:\microblog`，在这个目录下安装express（express代码会安装在这个目录下）：
<pre><code>D:\microblog>npm install -g express
</code></pre>

### step2. 生成工程 ###

express已经把项目生成移除了，要自动生成项目需要安装这个组件：
<pre><code>D:\microblog>npm install -g express-generator
</code></pre>

然后就可以运行express生成工程了，也就是，在你指定的目录下产生基本代码框架。在此之前，我们可以运行一下`express -h`，看看express命令的帮助。
<pre><code>Usage: express [options] [dir]

  Options:

    -h, --help          output usage information
    -V, --version       output the version number
    -e, --ejs           add ejs engine support (defaults to jade)
        --hbs           add handlebars engine support
    -H, --hogan         add hogan.js engine support
    -c, --css <engine>  add stylesheet <engine> support (less|stylus|compass) (d
efaults to plain css)
        --git           add .gitignore
    -f, --force         force on non-empty directory
</code></pre>
express帮助说明-e，添加ejs支持（默认是jade）。我们要用ejs，所以执行命令如下：
<pre><code>D:\microblog>express -e microblog
</code></pre>

### step3. 运行这个框架 ###
很不幸，express生成的这个框架不能如我们所愿直接用运行。还要做些工作。不过我们可以先尝试一下，看看错误结果：

	D:\microblog\microblog>node app.js

得到结果：
<pre><code>module.js:340
    throw err;
          ^
Error: Cannot find module 'basic-auth'
    at Function.Module._resolveFilename (module.js:338:15)
    at Function.Module._load (module.js:280:25)
    at Module.require (module.js:364:17)
    at require (module.js:380:17)
    at Object.<anonymous> (D:\microblog\microblog\node_modules\morgan\index.js:1
5:12)
    at Module._compile (module.js:456:26)
    at Object.Module._extensions..js (module.js:474:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Module.require (module.js:364:17)
</code></pre>
提示找不到`basic-auth`,应该还会找不到`depd`和`on-finished`。一次在我们的工程目录下执行：
<pre><code>D:\microblog\microblog>cnpm install basic-auth 
D:\microblog\microblog>cnpm install depd 
D:\microblog\microblog>cnpm install on-finished 
</code></pre>
然后执行：
<pre><code>D:\microblog\microblog>node app.js
</code></pre>
想想中的，程序一直运行，接受http请求没有出现，而是命令就这么一闪而过。打开入口文件`app.js`，找到代码：
<pre><code>app.use('/', routes);
app.use('/users', users);
</code></pre>
在其下增加一行代码：
<pre><code>app.listen(3000);
console.log('begin listen');
</code></pre>
奇迹出现了，程序开始*一直*运行了，打开浏览器，输入`http://localhost:3000/`或`http://127.0.0.1:3000/`。可以访问页面了！

如果想停止程序怎么办？`ctrl+c`！

### step4. Hello World！ ###
或者，我们运行点有用的功能，例如，通过你的网页获取你系统本地时间。打开`D:\microblog\microblog\routes\index.js`（express的路由分发），增加如下代码：
<pre><code>router.get('/hello', function(req, res, next) {
	res.send('The time is' + new Date().toString());
});
</code></pre>
访问`http://localhost:3000/hello`就可以访问我们制作的第一个网页了。

### step5. 使用以下ejs模板 ###
同样打开`D:\microblog\microblog\routes\index.js`。添加代码：

	router.get('/list', function(req, res, next) {
		res.render('list', {
			title: 'List',
			items: [1991, 'hello', 'express', 'Node.js']
		})
	});

打开`D:\microblog\microblog\views`，增加一个文件`listitem.ejs`,增加一行代码：

	<li><%= listitem %></li>

同样目录，增加文件`list.ejs`，增加一行代码：

	<% items.forEach(function(listitem){%><%include listitem %><%})%>
如果同时看书的同学应该知道：因为partial从ejs 3.0不能用了。这里我们改用forEach和include。

### step6. 开始microblog ###
打开`D:\microblog\microblog\routes\index.js`（可以注释掉我们前几步加入的内容）。增加如下代码：

	router.get('/', function(req, res, next) {
		res.render('index', { title: 'Express' });
	});
	
	router.get('/u/:user', function(req, res, next) {
	});
	
	router.post('/post', function(req, res, next) {
	});
	
	router.get('/reg', function(req, res, next) {
	});
	
	router.post('/reg', function(req, res, next) {
	});
	
	router.get('/login', function(req, res, next) {
	});
	
	router.post('/login', function(req, res, next) {
	});
	
	router.get('/logout', function(req, res, next) {
	});
下载Bootstrap，解压后将`img`目录内容放在`D:\microblog\microblog\public\images`中，`bootstrap.js`放入`D:\microblog\microblog\public\javascripts`，`bootstrap.css`和`bootstrap-responsive.css`放入`D:\microblog\microblog\public\stylesheets`，`bootstrap.js`放入`D:\microblog\microblog\public\javascripts`。

下载jquery，解压后放入`D:\microblog\microblog\public\javascripts`。
`D:\microblog\microblog\views`增加两个文件，一个是`header.ejs`：

	<!DOCTYPE html>
	<html>
		<head>
			<title><%= title %> - Microblog</title>
			<link rel='stylesheet' href='/stylesheets/bootstrap.css' />
			<style type="text/css">
				body {
					padding-top: 60px;
					padding-bottom: 40px;
				}
			</style>
			<link href="stylesheets/bootstrap-responsive.css" rel="stylesheet">
		</head>
		<body>
			<div class="navbar navbar-fixed-top">
				<div class="navbar-inner">
					<div class="container">
						<a class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
							<span class="icon-bar"></span>
							<span class="icon-bar"></span>
							<span class="icon-bar"></span>
						</a>
						<a class="brand" href="/">Microblog</a>
						<div class="nav-collapse">
							<ul class="nav">
								<li class="active"><a href="/">首页</a></li>
								<% if(!user) { %>
								<li><a href="/login">登入</a></li>
								<li><a href="/reg">注册</a></li>
								<% } else { %>
								<li><a href="/logout">登出</a></li>
								<% } %>
							</ul>
						</div>
					</div>
				</div>
			</div>
			<div id="container" class="container">
				<% if(success) { %>
				<div class="alert alert-success">
					<%= success %>
				</div>
				<% } %>
				<% if(error) { %>
				<div class="alert alert-error">
					<%= error %>
				</div>
				<% } %>

一个是`footer.ejs`：

			<footer>
				<p><a href="http://www.byvoid.com/" target="_blank">BYVoid</a> 2012</p>
			</footer>
			</div>
		</body>
		<script src="/javascripts/jquery.js"></script>
		<script src="/javascripts/bootstrap.js"></script>
	</html>

`index.ejs`内容改为如下：

	<%- include header %>
	<div class="hero-unit">
	<h1>欢迎来到 Microblog</h1>
	<p>Microblog 是一个基于 Node.js 的微博系统。</p>
	<p>
	<a class="btn btn-primary btn-large" href="/login">登录</a>
	<a class="btn btn-large" href="/reg">立即注册</a>
	</p>
	</div>
	<div class="row">
	<div class="span4">
	<h2>Carbo 说</h2>
	<p>东风破早梅 向暖一枝开 冰雪无人见 春从天上来</p>
	</div>
	<div class="span4">
	<h2>BYVoid 说</h2>
	<p>
	Open Chinese Convert（OpenCC）是一个开源的中文简繁转换项目，
	致力于制作高质量的基于统计预料的简繁转换词库。
	还提供函数库(libopencc)、命令行简繁转换工具、人工校对工具、词典生成程序、
	在线转换服务及图形用户界面。</p>
	</div>
	<div class="span4">
	<h2>佛振 说</h2>
	<p>中州韵输入法引擎 / Rime Input Method Engine 取意历史上通行的中州韵，
	愿写就一部汇集音韵学智慧的输入法经典之作。
	项目网站设在 http://code.google.com/p/rimeime/
	创造应用价值是一方面，更要坚持对好技术的追求，希望能写出灵动而易于扩展的代码，
	使其成为一款个性十足的开源输入法。</p>
	</div>
	</div>
	<%- include footer %>

执行：

	D:\microblog\microblog>node app.js

访问`http://localhost:3000/index`看看。

如果有看书的同学，应该知道：ejs 3.*之后不再支持layout.ejs的方法。所以我们在这里用的是`include`来辅助产生布局。有人评论`include`方法破坏了页面片段完整性，也有人说这是标准做法。

### step7. mongodb ###
在node中安装mongodb driver，修改package.json 如下：

	{
	  "name": "microblog",
	  "version": "0.0.0",
	  "private": true,
	  "scripts": {
	    "start": "node ./bin/www"
	  },
	  "dependencies": {
	    "body-parser": "~1.10.2",
	    "cookie-parser": "~1.3.3",
	    "debug": "~2.1.1",
	    "ejs": "~2.2.3",
	    "express": "~4.11.1",
	    "morgan": "~1.5.1",
	    "serve-favicon": "~2.2.0",
	 	"mongodb": "~1.4.30"
		"connect-mongo": "~0.7.0",
		"express-session": "~1.10.2",
	  }
	}

在package.json同级目录下运行：

	cnpm install

app.js同级目录下创建`setting.js`：

	module.exports = {
		cookieSecret: 'microblog',
		db: 'microblog',
		host: 'localhost',
	};

新建models文件夹，创建`db.js`文件：

	var settings = require('../settings');
	var Db = require('mongodb').Db;
	var Connection = require('mongodb').Connection;
	var Server = require('mongodb').Server;
	
	module.exports = new Db(settings.db, 
						   new Server(settings.host, Connection.DEFAULT_PORT, 
									  {}), 
									  {safe: true});

`app.js`中增加：

	var session = require('express-session');
	var connectMongo = require('connect-mongo')(session);
	var settings = require('./settings')；
	
	app.use(session({
	 secret: settings.cookieSecret,
	 store: new connectMongo({
	  db: settings.db,
	 })
	}));

启动一个新的cmd，执行：

	D:\>mongod --dbpath d:\microblog\microblog\mongodb\db

启动mongodb，然后启动服务器，应该已经成功了。但是会报一个连接数据库时的警告，有碍观瞻。解决方法是修改`D:\microblog\microblog\models\db.js`：

	module.exports = new Db(setting.db, new Server(setting.host, Connection.DEFAULT_PORT), {safe:true});

### step8. post访问数据库会报错 ###

	collection.ensureIndex('user', function(err, post){});
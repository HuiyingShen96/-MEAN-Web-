[TOC]
###1. Express安装
`$ npm install express`

-------------------------------------------------
###2. 创建第一个Express应用
1\. 创建package.json文件
`$ npm init`
2\. 新建server.js文件，其代码如下：
```javascript
var express = require('express'); // 引用Express模块
var app = express(); // 创建一个Express对象
app.use('/', function(req, res) { // 用app.use()在指定路径加载一个中间件函数
   res.send('Hello World');
});
app.listen(3000); // 用app.listen()方法设置应用需要监听的3000端口
console.log('Server running at http://localhost:3000/');
module.exports = app; // module.exports对象是用于返回应用程序对象的
```
3\. 运行新创建的应用
`$ node server`

-------------------------------------------------
###3. 应用、请求和响应对象
*应用对象*是指上面例子中创建的Express应用实例，通常它会被用于实现对应用的配置。
*请求对象*是指Node HTTP请求对象的封装，用于获取当前正在处理的请求信息。
*响应对象*是指Node HTTP响应对象的封装，用于设置响应包头和数据。

**应用对象**
- `app.set(name, value)`: 用于设置Express配置中的环境变量
- `app.get(name)`: 用于获取Express配置中的环境变量
- `app.engine(ext, callback)`: 用于指定模板引擎中的渲染文件类型
- `app.locals`: 用于向渲染模板发送应用级变量
- `app.use([path], callback)`:用于创建处理HTTP请求的中间件
- `app.VERB(path, [callback...], callback)`: 用于定义一个或多个中间件函数，以响应特定HTTP方法发往某一指定路径的请求
- `app.route(path).VERB([callback...], callback)`: 用于定义一个或多个中间件函数，以响应发往某一路径的多种HTTP请求方法
- `app.param([name], callback)`: 用于发往某一路径且包含指定路由参数的请求附加某一特定功能

**请求对象**
- `req.query`: 即已解析为对象的所有GET参数
- `req.params`: 即已解析为对象的路由参数
- `req.body`: 该属性包含在bodyParser()中间件中，用于获取所有请求的body部分
- `req.param(name)`: 用于获取请求参数，包括GET参数、路由参数，或请求body部分的JSON属性
- `req.path、req.host及req.ip`: 即当时请求的路径、主机名和访问者的IP
- `req.cookies`: 该方法需要与cookieParser()中间件结合使用，用于获取用户浏览器传来的cookies

**响应对象**
- `res.status(code)`:用于设置响应的HTTP状态码
- `res.set(field, [value])`: 用于设置响应的HTTP的报头
- `res.cookie(name, value, [options])`: 用于设置响应的浏览器cookies（options参数是一个对象，用于定义cookies配置）
- `res.redirect([status], url)`: 用于将请求重新定位到参数url所定义的URL（HTTP状态码是可添加的）
- `res.send([body | status], [body])`: 主要用于非流式响应
- `res.json([status | body], [body])`: 若用于发送对象或数组，该方法与res.send()完全一致。不过大多数情况下，该方法将会被用作为语法糖。然后某些情况下，它也可以被用来轻质发送JSON的空对象，比如null和undefined。
- `res.render(view, [locals], callback)`: 用于视图渲染，并发送包含HTML的响应

-------------------------------------------------
###4. 组织项目结构
#####1. 水平文件夹结构

#####2. 垂直文件夹结构

#####3. 文件命名约定
A simple approach would be to name our Express controller *feature.server.controller.js*
and our AngularJS controller *feature.client.controller.js*.
#####4. 实践水平文件夹结构
1\. 创建项目文件夹结构如下

2\. 在根目录创建package.json文件如下：
```json
{
    "name" : "MEAN",
    "version" : "0.0.3",
    "dependencies" : {
        "express" : "~4.8.8"
    }
}
```
3\. 创建控制器：在app/controllerss文件夹下创建index.server.controller.js文件：
```javascript
exports.render = function(req, res) {
   res.send('Hello World');
};
```
4\. 创建Express路由功能：在app/routes文件夹下创建index.server.routes.js文件：
```javascript
module.exports = function(app) {
   var index = require('../controllers/index.server.controller');
   app.get('/', index.render);
};
```
5\. 创建Express对象：在app/config文件夹下创建express.js文件
express.js专门用于配置Express应用，所有与Express应用相关的配置也需要添加到这个文件中。
```javascript
var express = require('express');
module.exports = function() {
   var app = express();
   require('../app/routes/index.server.routes.js')(app); // 调用前面创建的路由文件，以函数调用的方式传入了应用实例
   return app;
};
```
6\. 在根目录下创建server.js文件
```javascript
var express = require('./config/express');

var app = express();
app.listen(3000);
module.exports = app;

console.log('Server running at http://localhost:3000/');
```
7\. 安装依赖和启动应用
用命令行工具进入到应用程序根目录：
`$ node install`
`$ node server`

-----------------------------------------------
###5. Express应用配置
#####1. 根据运行环境来配置应用
1\. 添加依赖：编辑package.json文件如下：
```json
{
    "name": "MEAN",
    "version": "0.0.3",
   "dependencies": {
      "express": "~4.8.8",
      "morgan": "~1.3.0",
      "compression": "~1.0.11",
      "body-parser": "~1.8.0",
      "method-override": "~2.2.0"
   }
}
```
2\. 修改config/express.js文件
```javascript
var express = require('express'),
   morgan = require('morgan'), // 提供简单的日志中间件
   compress = require('compression'), // 提供响应内容的压缩功能
   bodyParser = require('body-parser'), // 包含几个处理请求数据的中间件
   methodOverride = require('method-override'); // 提供了对HTTP DELETE和PUT两个遗留方法的支持
module.exports = function() {
   var app = express();
   if (process.env.NODE_ENV === 'development') {  // 对系统环境进行判定
       app.use(morgan('dev'));
   } else if (process.env.NODE_ENV === 'production') {
       app.use(compress());
   }
   app.use(bodyParser.urlencoded({
       extended: true
   }));
   app.use(bodyParser.json());
   app.use(methodOverride());
   require('../app/routes/index.server.routes.js')(app);
   return app;
};
```
3\. 对server.js作出相应修改
在文件开始的地方添加如下代码：
```javascript
process.env.NODE_ENV = process.env.NODE_ENV || 'development'; // 默认值设为development
```

#####2. 环境配置文件
可以使用环境配置文件对不同的配置进行管理。下面为默认的开发环境创建一个配置文件。
1\. 在config/env文件夹中创建一个development.js文件：
```javascript 
module.exports = {
   // Development configuration options
};
```
2\. 管理配置文件的加载
在config目录下创建一个config.js文件：
```javascript
module.exports = require('./env/' + process.env.NODE_ENV + '.js');
```
-----------------------------------------
###6. Express路由机制
---------------------------------------------
###7. 渲染视图
渲染视图：将具体的数据传入模板引擎，再由引擎渲染出最终视图。
Express有两种方法渲染：
1. 使用app.render()，将HTML交由一个回调函数进行渲染
2. 【更常用】使用res.render()，将视图渲染成HTML后直接作为响应输出
#####1. 配置视图系统（以EJS模板引擎为例）
1\. 添加依赖`npm install ejs --save`
2\. 将EJS设置为默认的模板引擎
修改config/express.js：
```javascript
...
app.use(methodOverride());

app.set('views', './app/views'); // 设置视图文件的存储目录
app.set('view engine', 'ejs'); // 设置EJS作为Express应用的模板引擎

require('../app/routes/index.server.routes.js')(app);
...
```
#####2. EJS视图渲染
1\. 在app/views目录下创建一个index.ejs文件：
```html
<!DOCTYPE html>
<html>
<head>
   <title><%= title %></title>
</head>
<body>
   <h1><%= title %></h1>
</body>
</html>
```
2\. 配置控制器去渲染模板并自动将其转换为HTML响应输出
修改app/controllers/index.server.controller.js文件如下：
```javascript
exports.render = function(req, res) {
   res.render('index', {
       title: 'Hello World'
   })
};
```
res.render()方法：
第一个参数是EJS模板文件名中去掉扩展名的部分，
第二个参数是包含有模板变量的对象。

---------------------------------------
###8. 静态文件服务
Express通过预置的express.static()中间件来提供静态文件服务。
如下修改config/express.js：
```javascrilpt
require('../app/routes/index.server.routes.js')(app);

app.use(express.static('./public')); // 参数用于指定静态文件所在的文件夹路径

return app;
```
该中间件启动的位置，位于路由中间件之下，即先执行路由逻辑。

*测试例子*：
在public/img中增加一张名为logo.png的图片，然后如下修改app/views/index.ejs文件：
```html
<body>
   <img src="img/logo.png" alt="Logo">
   <h1><%= title %></h1>
</body>
```

-------------------------------------
###9. 配置会话
会话的常见功能是对Web应用方可的行为进行跟踪。
1\. 安装express-session中间件`$ npm install express-session --save-dev`
2\. 设置cookie密钥
express-seexion模块通过浏览器的cookie来存储用户的唯一标识。为了标记会话，需要使用一个密钥，这可以有效防止恶意的会话污染。
为了安全起见，建议在不同的环境中使用不同的cookie密钥，这就涉及根据环境加载不同的配置文件。
此处只设置development环境下的cookie密钥。
修改config/env/development.js文件：
```javascript
module.exports = {
   sessionSecret: 'developmentSessionSecret'
};
```
3\. 使用上述配置文件对Express应用进行配置
修改config/express.js文件：
```javascript
var config = require('./config'),
...
app.use(methodOverride());

app.use(session({
   saveUninitialized: true,
   resave: true,
   secret: config.sessionSecret
}));

app.set('views', './app/views');
...
```
session中间件会为应用中所有的请求对象增加一个session对象，通过这个session对象，可以设置或者获取当前会话的任意属性。
4\. 测试例子：
如下修改app/controller/index.server.controller.js文件：
```javascript
exports.render = function(req, res){
    var title = "";
    if (req.session.isVisit) {
        req.session.isVisit++;
        title = "Welcome your "+ req.session.isVisit + "th visit!"
    }else{
        req.session.isVisit = 1;
        title = 'Welcome for the first time here!';
    }
    res.render('index', {
        title: title
    })
};
```
---------------------
###10. 附录
[Express Official Website](http://expressjs.com/)
[Express官网（中文）](http://www.expressjs.com.cn/)
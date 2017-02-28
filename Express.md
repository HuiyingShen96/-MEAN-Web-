[TOC]
###1. Express��װ
`$ npm install express`

-------------------------------------------------
###2. ������һ��ExpressӦ��
1\. ����package.json�ļ�
`$ npm init`
2\. �½�server.js�ļ�����������£�
```javascript
var express = require('express'); // ����Expressģ��
var app = express(); // ����һ��Express����
app.use('/', function(req, res) { // ��app.use()��ָ��·������һ���м������
   res.send('Hello World');
});
app.listen(3000); // ��app.listen()��������Ӧ����Ҫ������3000�˿�
console.log('Server running at http://localhost:3000/');
module.exports = app; // module.exports���������ڷ���Ӧ�ó�������
```
3\. �����´�����Ӧ��
`$ node server`

-------------------------------------------------
###3. Ӧ�á��������Ӧ����
*Ӧ�ö���*��ָ���������д�����ExpressӦ��ʵ����ͨ�����ᱻ����ʵ�ֶ�Ӧ�õ����á�
*�������*��ָNode HTTP�������ķ�װ�����ڻ�ȡ��ǰ���ڴ����������Ϣ��
*��Ӧ����*��ָNode HTTP��Ӧ����ķ�װ������������Ӧ��ͷ�����ݡ�

**Ӧ�ö���**
- `app.set(name, value)`: ��������Express�����еĻ�������
- `app.get(name)`: ���ڻ�ȡExpress�����еĻ�������
- `app.engine(ext, callback)`: ����ָ��ģ�������е���Ⱦ�ļ�����
- `app.locals`: ��������Ⱦģ�巢��Ӧ�ü�����
- `app.use([path], callback)`:���ڴ�������HTTP������м��
- `app.VERB(path, [callback...], callback)`: ���ڶ���һ�������м������������Ӧ�ض�HTTP��������ĳһָ��·��������
- `app.route(path).VERB([callback...], callback)`: ���ڶ���һ�������м������������Ӧ����ĳһ·���Ķ���HTTP���󷽷�
- `app.param([name], callback)`: ���ڷ���ĳһ·���Ұ���ָ��·�ɲ��������󸽼�ĳһ�ض�����

**�������**
- `req.query`: ���ѽ���Ϊ���������GET����
- `req.params`: ���ѽ���Ϊ�����·�ɲ���
- `req.body`: �����԰�����bodyParser()�м���У����ڻ�ȡ���������body����
- `req.param(name)`: ���ڻ�ȡ�������������GET������·�ɲ�����������body���ֵ�JSON����
- `req.path��req.host��req.ip`: ����ʱ�����·�����������ͷ����ߵ�IP
- `req.cookies`: �÷�����Ҫ��cookieParser()�м�����ʹ�ã����ڻ�ȡ�û������������cookies

**��Ӧ����**
- `res.status(code)`:����������Ӧ��HTTP״̬��
- `res.set(field, [value])`: ����������Ӧ��HTTP�ı�ͷ
- `res.cookie(name, value, [options])`: ����������Ӧ�������cookies��options������һ���������ڶ���cookies���ã�
- `res.redirect([status], url)`: ���ڽ��������¶�λ������url�������URL��HTTP״̬���ǿ���ӵģ�
- `res.send([body | status], [body])`: ��Ҫ���ڷ���ʽ��Ӧ
- `res.json([status | body], [body])`: �����ڷ��Ͷ�������飬�÷�����res.send()��ȫһ�¡��������������£��÷������ᱻ����Ϊ�﷨�ǡ�Ȼ��ĳЩ����£���Ҳ���Ա��������ʷ���JSON�Ŀն��󣬱���null��undefined��
- `res.render(view, [locals], callback)`: ������ͼ��Ⱦ�������Ͱ���HTML����Ӧ

-------------------------------------------------
###4. ��֯��Ŀ�ṹ
#####1. ˮƽ�ļ��нṹ

#####2. ��ֱ�ļ��нṹ

#####3. �ļ�����Լ��
A simple approach would be to name our Express controller *feature.server.controller.js*
and our AngularJS controller *feature.client.controller.js*.
#####4. ʵ��ˮƽ�ļ��нṹ
1\. ������Ŀ�ļ��нṹ����

2\. �ڸ�Ŀ¼����package.json�ļ����£�
```json
{
    "name" : "MEAN",
    "version" : "0.0.3",
    "dependencies" : {
        "express" : "~4.8.8"
    }
}
```
3\. ��������������app/controllerss�ļ����´���index.server.controller.js�ļ���
```javascript
exports.render = function(req, res) {
   res.send('Hello World');
};
```
4\. ����Express·�ɹ��ܣ���app/routes�ļ����´���index.server.routes.js�ļ���
```javascript
module.exports = function(app) {
   var index = require('../controllers/index.server.controller');
   app.get('/', index.render);
};
```
5\. ����Express������app/config�ļ����´���express.js�ļ�
express.jsר����������ExpressӦ�ã�������ExpressӦ����ص�����Ҳ��Ҫ��ӵ�����ļ��С�
```javascript
var express = require('express');
module.exports = function() {
   var app = express();
   require('../app/routes/index.server.routes.js')(app); // ����ǰ�洴����·���ļ����Ժ������õķ�ʽ������Ӧ��ʵ��
   return app;
};
```
6\. �ڸ�Ŀ¼�´���server.js�ļ�
```javascript
var express = require('./config/express');

var app = express();
app.listen(3000);
module.exports = app;

console.log('Server running at http://localhost:3000/');
```
7\. ��װ����������Ӧ��
�������й��߽��뵽Ӧ�ó����Ŀ¼��
`$ node install`
`$ node server`

-----------------------------------------------
###5. ExpressӦ������
#####1. �������л���������Ӧ��
1\. ����������༭package.json�ļ����£�
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
2\. �޸�config/express.js�ļ�
```javascript
var express = require('express'),
   morgan = require('morgan'), // �ṩ�򵥵���־�м��
   compress = require('compression'), // �ṩ��Ӧ���ݵ�ѹ������
   bodyParser = require('body-parser'), // �������������������ݵ��м��
   methodOverride = require('method-override'); // �ṩ�˶�HTTP DELETE��PUT��������������֧��
module.exports = function() {
   var app = express();
   if (process.env.NODE_ENV === 'development') {  // ��ϵͳ���������ж�
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
3\. ��server.js������Ӧ�޸�
���ļ���ʼ�ĵط�������´��룺
```javascript
process.env.NODE_ENV = process.env.NODE_ENV || 'development'; // Ĭ��ֵ��Ϊdevelopment
```

#####2. ���������ļ�
����ʹ�û��������ļ��Բ�ͬ�����ý��й�������ΪĬ�ϵĿ�����������һ�������ļ���
1\. ��config/env�ļ����д���һ��development.js�ļ���
```javascript 
module.exports = {
   // Development configuration options
};
```
2\. ���������ļ��ļ���
��configĿ¼�´���һ��config.js�ļ���
```javascript
module.exports = require('./env/' + process.env.NODE_ENV + '.js');
```
-----------------------------------------
###6. Express·�ɻ���
---------------------------------------------
###7. ��Ⱦ��ͼ
��Ⱦ��ͼ������������ݴ���ģ�����棬����������Ⱦ��������ͼ��
Express�����ַ�����Ⱦ��
1. ʹ��app.render()����HTML����һ���ص�����������Ⱦ
2. �������á�ʹ��res.render()������ͼ��Ⱦ��HTML��ֱ����Ϊ��Ӧ���
#####1. ������ͼϵͳ����EJSģ������Ϊ����
1\. �������`npm install ejs --save`
2\. ��EJS����ΪĬ�ϵ�ģ������
�޸�config/express.js��
```javascript
...
app.use(methodOverride());

app.set('views', './app/views'); // ������ͼ�ļ��Ĵ洢Ŀ¼
app.set('view engine', 'ejs'); // ����EJS��ΪExpressӦ�õ�ģ������

require('../app/routes/index.server.routes.js')(app);
...
```
#####2. EJS��ͼ��Ⱦ
1\. ��app/viewsĿ¼�´���һ��index.ejs�ļ���
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
2\. ���ÿ�����ȥ��Ⱦģ�岢�Զ�����ת��ΪHTML��Ӧ���
�޸�app/controllers/index.server.controller.js�ļ����£�
```javascript
exports.render = function(req, res) {
   res.render('index', {
       title: 'Hello World'
   })
};
```
res.render()������
��һ��������EJSģ���ļ�����ȥ����չ���Ĳ��֣�
�ڶ��������ǰ�����ģ������Ķ���

---------------------------------------
###8. ��̬�ļ�����
Expressͨ��Ԥ�õ�express.static()�м�����ṩ��̬�ļ�����
�����޸�config/express.js��
```javascrilpt
require('../app/routes/index.server.routes.js')(app);

app.use(express.static('./public')); // ��������ָ����̬�ļ����ڵ��ļ���·��

return app;
```
���м��������λ�ã�λ��·���м��֮�£�����ִ��·���߼���

*��������*��
��public/img������һ����Ϊlogo.png��ͼƬ��Ȼ�������޸�app/views/index.ejs�ļ���
```html
<body>
   <img src="img/logo.png" alt="Logo">
   <h1><%= title %></h1>
</body>
```

-------------------------------------
###9. ���ûỰ
�Ự�ĳ��������Ƕ�WebӦ�÷��ɵ���Ϊ���и��١�
1\. ��װexpress-session�м��`$ npm install express-session --save-dev`
2\. ����cookie��Կ
express-seexionģ��ͨ���������cookie���洢�û���Ψһ��ʶ��Ϊ�˱�ǻỰ����Ҫʹ��һ����Կ���������Ч��ֹ����ĻỰ��Ⱦ��
Ϊ�˰�ȫ����������ڲ�ͬ�Ļ�����ʹ�ò�ͬ��cookie��Կ������漰���ݻ������ز�ͬ�������ļ���
�˴�ֻ����development�����µ�cookie��Կ��
�޸�config/env/development.js�ļ���
```javascript
module.exports = {
   sessionSecret: 'developmentSessionSecret'
};
```
3\. ʹ�����������ļ���ExpressӦ�ý�������
�޸�config/express.js�ļ���
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
session�м����ΪӦ�������е������������һ��session����ͨ�����session���󣬿������û��߻�ȡ��ǰ�Ự���������ԡ�
4\. �������ӣ�
�����޸�app/controller/index.server.controller.js�ļ���
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
###10. ��¼
[Express Official Website](http://expressjs.com/)
[Express���������ģ�](http://www.expressjs.com.cn/)
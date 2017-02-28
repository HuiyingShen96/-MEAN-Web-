[TOC]
>Mongoose is a robust Node.js ODM module that adds MongoDB support to your Express application.
>The Mongoose design goal is to bridge the gap between the MongoDB schemaless approach and the requirements of real-world application development.
###0. 使用前
1. 安装Mongoose
`$ npm install mongoose --save-dev`
2. 连接MongoDB
MongoDB字符串：一个URL，用于为MongoDB驱动指定需要连接的数据库实例，其构造如下：
`mongodb://username:password@hostname:port/database`
如果需要连接的是本地服务器，则可以简写为：`mongodb://localhost/mean-book`
使用例子：
```javascript
var uri = 'mongodb://localhost/mean-book';
var db = require('mongoose').connect(uri);
```
最佳方案：
(1) 将应用程序变量存在环境配置文件中。修改config/env/development.js如下：
```javascript
module.exports = {
   db: 'mongodb://localhost/mean-book',
   sessionSecret: 'developmentSessionSecret'
};
```
(2) 然后在config目录下创建mongoose.js，代码如下：
```javascript
var config = require('./config'),
mongoose = require('mongoose');

module.exports = function() {
   var db = mongoose.connect(config.db);
   return db;
};
```
(3) 通过修改server.js文件对Mongoose的配置进行初始化
```javascript
process.env.NODE_ENV = process.env.NODE_ENV || 'development';

var mongoose = require('./config/mongoose'), // 注意Mongoose配置文件必须是server.js中第一个加载的配置文件，以便在Mongoose加载完成后，任何模块无需加载便可以直接使用Mongoose模型。
express = require('./config/express');

var db = mongoose();
var app = express();
app.listen(3000);

module.exports = app;

console.log('Server running at http://localhost:3000/');
```

###1. Mongoose的模式与模型
Mongoose使用模式对象定义文档的各个属性，每个属性都有其类型和约束，以便控制文档的结构。
#####1. 创建User模式与模型
在app/models文件夹下创建user.server.model.js文件，代码如下：
```javascript
var mongoose = require('mongoose'),
   Schema = mongoose.Schema;

var UserSchema = new Schema({
   firstName: String,
   lastName: String,
   email: String,
   username: String,
   password: String
});

mongoose.model('User', UserSchema);
```
#####2. 注册User模型
在config/mongoose.js文件中包含user.server.model.js文件：
```javascript
...
module.exports = function() {
   var db = mongoose.connect(config.db);
   require('../app/models/user.server.model');
   return db;
};
```


-------------------------------------------------------------------
###2. 模式的索引、修饰符和虚拟属性
#####1. 定义默认值
如，在UserSchema中增加一个created的时间字段来保存用户注册的时间。该字段将在随想创建时对创建时间进行初始化保存。
编辑app/models/user.server.model.js文件：
```javascript
   created:{
        type: Date,
        default: Date.now
    },
```
#####2. 使用模式修饰符
1\. 预定义修饰符
Mongoose预定义了一些简单的修饰符，如对字符类型的字段使用trim修饰符去除两端的空格，使用uppercase修饰符转换为大写字母等。
使用如下：
```javascript
    username: {
        type: String,
        trim: true, //确保字段的开头和末尾不会有空格
    },
```
2\. 自定义setter修饰符
自定义setter修饰符用于在保存文档前执行数据操作
```javascript
    website: {
        type: String,
        set: function(url){
            if(!url){
                return url;
            }else if(url.indexOf('http://') !== 0 && url.indexOf('https://' !== 0)){
                url = 'http://' + url;
                return url;
            }
        }
    },
```
3\. 自定义getter修饰符
自定义getter修饰符用于在讲文档向下级进行输出前对文档数据进行修改。
```javascript
var UserSchema = new Schema({
...
    website: {
        type: String,
        get: function(url){
            if(!url){
                return url;
            }else if(url.indexOf('http://') !== 0 && url.indexOf('https://' !== 0)){
                url = 'http://' + url;
                return url;
            }
        }
    },
...
});
UserSchema.set('toJSON', { getters: true });
```
因为在res.json()等方法中，文档转换为JSON默认不会执行getter修饰符的操作，所以这里调用了UserSchema.set()，以便保证在这种情况下强制执行getter修饰符。
4\. [更多相关信息请戳我](http://mongoosejs.com/docs/api.html)

#####3. 增加虚拟属性
有时我们会需要一些动态计算的文档属性，但又不需要将它们真正存储到文档中，这时便可以使用虚拟属性。
```javascript
UserSchema.virtual('fullName').get(function(){
    return this.firstName + ' ' + this.lastName;
});
UserSchema.set('toJSON', { getters: true, virtuals: true });
```
But virtual attributes can also have **setters** to help you save your documents as you prefer instead of just adding more field attributes. In this case, let's say you wanted to break an input's fullName field into your first and last name fields. To do so, a modified virtual declaration would look like the following code snippet:
```javascript
UserSchema.virtual('fullName').get(function(){
    return this.firstName + ' ' + this.lastName;
}).set(function(fullName){
    var splitName = fullName.split(' ');
    this.firstName = splitName[0] || '';
    this.lastName = splitName[1] || '';
});
```
#####4. 使用索引优化查询（谨慎使用）
1\. 唯一索引`unique: true`
```javascript
    username: {
        type: String,
        trim: true, 
        unique: true
    },
```
2\. 辅助索引
```javascript
    email: {
        type: String,
        index: true
    },
```

-------------------------------------------------------------------
###3. 使用模型的方法处理增删改查操作
#####1. 使用save()创建新文档
(1) 创建一个Users控制器，用来处理所有与用户相关的操作
在app/controller目录下创建users.server.controller.js文件，代码如下：
```javascript
var User = require('mongoose').model('User');
exports.create = function(req, res, next) {
   var user = new User(req.body);
   user.save(function(err) {
       if (err) {
           return next(err);
       } else {
           res.json(user);
       }
   });
};
```
(2) 使用POST方法注册路由
在app/routes目录中创建users.server.routes.js文件，代码如下：
```javascript
var users = require('../../app/controllers/users.server.controller');
module.exports = function(app) {
   app.route('/users')
   .post(users.create); 
};
```
(3) 引用路由文件
修改config/express.js如下：
```javascript
require('../app/routes/users.server.routes.js')(app);
```
(4) 测试
（可以使用chrome的“基于REST的Web服务客户端”扩展程序）
使用HTTP的POST方法请求users的基础路径，请求报体包含如下的JSON数据：
```json
{
   "firstName": "First",
   "lastName": "Last",
   "email": "user@example.com",
   "username": "username",
   "password": "password"
}
```
#####2. 使用find()查找多个文档
(1) 在app/controllers/users.server.controller.js文件中增加一个list()方法：
```javascript
exports.list = function(req, res, next) {
   User.find({}, function(err, users) {
       if (err) {
           return next(err);
       } else {
           res.json(users);
       }
   });
};
```
(2) 使用GET方法注册路由：
修改app/routes/users.server.routes.js文件如下：
```javascript
//other code
app.route('/users')
   .post(users.create)
   .get(users.list);
//other code
```
(3) 其它：find()的高级查找
find()的四个参数：
- Query: MongoDB查询对象
- [Fields]:可选，返回指定的字段
- [Options]: 可选，查询配置选项对象
- [Callback]:可选，回调函数
[更多关于查询选项的信息请戳我](http://mongoosejs.com/docs/api.html)

#####3. 使用findOne()读取单个文档
findOne()只获取特定子集中的第一个文档
(1) 在app/controllers/users.server.controller.js文件中增加read()和userByID()方法：
```javascript
exports.read = function(req, res){
    res.json(req.user);
};

exports.userByID = function(req, res, next, id){
    User.findOne({ _id: id},function(err, user){
        if(err){
            return next(err);
        }else{
            req.user = user;
            next();
        }
    });
};
```
(2) 使用GET和app.param()方法注册路由
修改app/routes/users.server.routes.js文件如下：
```javascript
//other code
app.route('/users/:userId')
   .get(users.read);
app.param('userId', users.userByID);
//other code
```
app.param()定义的中间件负责生成req.user对象，会在任何 注册时使用userId参数的中间件 之前执行，即users.userById()方法会在users.read()这个中间件之前执行。
(3) 测试
使用`$ node server`命令运行应用，然后用浏览器访问http://localhost:3000/users/[id]，请注意访问前先用所选取的用户_id值将[id]替换一下。
#####4. 更新已有文档
方法包括：update()、findOneAndUpdate()、findByIdAndUpdate()等。
由于前面已经创建了userById()中间件，此处使用findByIdAndUpdate()作为例子。
(1) 在app/controllers/users.server.controller.js文件中增加update()方法：
```javascript
exports.update = function(req, res, next){
    User.findByIdAndUpdate(req.user.id, req.body, function(err, user){
        if(err){
            return next(err);
        }else{
            res.json(user);
        }
    });
};
```
(2) 使用PUT方法注册路由
```javascript
app.route('/users/:userId')
        .get(users.read)
        .put(users.update);
```
(3) 测试
请求方法：PUT
请求路径：localhost:3000/users/[id]
请求体：{'{"lastName":"Updated"}

#####5. 删除已有文档
方法包括：remove()、findOneAndRemove()、findByIdAndRemove()等
以下例子使用findByIdAndRemove()方法
(1) 在app/controllers/users.server.controller.js文件中增加delete()方法：
```javascript
exports.delete = function(req, res, next) {
    req.user.remove(function(err) {
        if (err) {
            return next(err);
        } else {
            res.json(req.user);
        }
    })
};
```
(2) 使用DELETE方法注册路由
```javascript
app.route('/users/:userId')
        .get(users.read)
        .put(users.update)
        .delete(users.delete);
```
(3) 测试
请求方法：DELETE
请求路径：localhost:3000/users/[id]
#####6. 模型方法自定义
1\. 自定义静态方法
模型静态方法定义在模式的statics属性中。
定义（在app/models/user.server.model.js文件中定义）：
```javascript
UserSchema.statics.findOneByUsername = function (username,callback) {
   this.findOne({ username: new RegExp(username, 'i') }, callback);
};
```
使用（在app/controllers/users.server.controller.js文件中使用）：
```javascript
User.findOneByUsername('username', function(err, user){
   ...
});
```
2\. 自定义实例方法
在模型文件中定义：
```javascript
UserSchema.methods.authenticate = function(password) {
   return this.password === password;
};
```
使用：
```javascript
var user = new User(req.body);
user.authenticate('password');
```

-------------------------------------------------------------------------------------
###4. 使用预定义和自定义的验证器来对数据进行校验
While you can validate your data at the logic layer of your application, it is more useful to do it at the model level. Luckily, Mongoose supports both simple predefined validators and more complex custom validators. Validators are defined at the field level of a document and are executed when the document is being saved. If a validation error occurs, the save operation is aborted and the error is passed to the callback.
1\. 预定义的验证器
required验证器：检查对应的值是否存在
match验证器：只有能被正则表达式匹配相应字段的文档才能保存
enum验证器：对字段的值域进行限定
[更多关于Mongoose预置验证器的信息请戳我](http://mongoosejs.com/docs/validation.html)
```javascript
var UserSchema = new Schema({
   ...
   username: {
       type: String,
       trim: true,
       unique: true,
       required: true
   },
   email: {
       type: String,
       index: true,
       match: /.+\@.+\..+/
   },
   role: {
       type: String,
       enum: ['Admin', 'Owner', 'User']
   },
   ...
});
```
2\. 自定义的验证器
要想自定义验证器，定义字段的validate属性即可。该属性是一个数组，数组包括一个函数和一个报错消息。
```javascript
var UserSchema = new Schema({
   ...
   password: {
       type: String,
       validate: [
           function(password) {
               return password.length >= 6;
           },
           'Password should be longer'
       ]
   },
...
});
```
###5. 使用中间件拦截处理模型方法
1\. 预处理中间件
预处理中间件在操作执行之前触发。For instance, a pre-save middleware will get executed before the saving of the document. This functionality makes pre middleware perfect for more complex validations and default values assignment.（复杂的验证器和默认值初始化功能）
使用模式对象的`pre()`方法即可定义与处理中间件。
```javascript
UserSchema.pre('save', function(next) {
   if (...) {
       next()
   } else {
       next(new Error('An Error Occured'));
   }
});
```
2\. 后置处理中间件
后置处理中间件在操作执行完成之后触发。For instance, a postsave middleware will get executed after saving the document. This functionality makes post middleware perfect to log your application logic.（日志功能）
后置处理中间件使用模式对象的`post()`方法进行定义。
```javascript
UserSchema.post('save', function(next) {
   if(this.isNew) {
       console.log('A new user was created.');
   } else {
       console.log('A user updated is details.');
   }
});
```
3\. [关于Mongoose中间件的更多信息请戳我](http://mongoosejs.com/docs/middleware.html)

-----------------------------------------
###6. 使用Mongoose DBRef
MongoDB是不支持连接查询的，但可以使用DBRef在不同的文档之间建立引用关系。Mongoose通过模式中的ObjectID类型及ref属性来实现对DBRef的支持。Mongoose还支持在查询时将子文档填充到父文档中。
例如我们要为博文创建一个PostSchema模式，每个博文都通过PostSchema的author字段来对应作者，这一字段由user模型的实例构成。PostSchema定义代码如下:
```javascript
var PostSchema = new Schema({
   title: {
       type: String,
       required: true
   },
   content: {
       type: String,
       required: true
   },
   author: {
       type: Schema.ObjectId,
       ref: 'User'
   }
});
mongoose.model('Post', PostSchema);
```
请注意，ref属性指定使用User模型来填充author字段。
在使用中，创建新的博文时，必须先通过检索或者创建的方式获取User模型实例，再用User实例作为博文author字段的值，代码如下:
```javascript
var user = new User();
user.save();

var post = new Post();
post.author = user;
post.save();    
```
post文档中将创建一个关联了引用文档的DBRef，检索时便可依此获取引用文档。
DBRef中只是存了引用文档的ObjectlD,  Mongoose还需要使用user实例来填充post实例。检索博文文档时，使用populate()方法便可以将用户文档填充过来。下面这段代码，演示了如何在find()的结果集中对author字段进行填充:
```javascript
Post.find().populate('author').exec(function(err, posts) {
   ...
});
```
上述代码将会检索整个post集合，所有文档的author字段会使用对应的user进行填充。
DBRef是MongoDB一个很棒的功能，Mongoose对这一功能的支持，使得模型可以通过对象引用实现更好的组织。本书后面的内容中，也会用DBRef来支撑应用逻辑。
[更多关于DBRef的信息请戳我](http://mongoosejs.com/docs/populate.html)












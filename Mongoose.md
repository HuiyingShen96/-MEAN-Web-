[TOC]
>Mongoose is a robust Node.js ODM module that adds MongoDB support to your Express application.
>The Mongoose design goal is to bridge the gap between the MongoDB schemaless approach and the requirements of real-world application development.
###0. ʹ��ǰ
1. ��װMongoose
`$ npm install mongoose --save-dev`
2. ����MongoDB
MongoDB�ַ�����һ��URL������ΪMongoDB����ָ����Ҫ���ӵ����ݿ�ʵ�����乹�����£�
`mongodb://username:password@hostname:port/database`
�����Ҫ���ӵ��Ǳ��ط�����������Լ�дΪ��`mongodb://localhost/mean-book`
ʹ�����ӣ�
```javascript
var uri = 'mongodb://localhost/mean-book';
var db = require('mongoose').connect(uri);
```
��ѷ�����
(1) ��Ӧ�ó���������ڻ��������ļ��С��޸�config/env/development.js���£�
```javascript
module.exports = {
   db: 'mongodb://localhost/mean-book',
   sessionSecret: 'developmentSessionSecret'
};
```
(2) Ȼ����configĿ¼�´���mongoose.js���������£�
```javascript
var config = require('./config'),
mongoose = require('mongoose');

module.exports = function() {
   var db = mongoose.connect(config.db);
   return db;
};
```
(3) ͨ���޸�server.js�ļ���Mongoose�����ý��г�ʼ��
```javascript
process.env.NODE_ENV = process.env.NODE_ENV || 'development';

var mongoose = require('./config/mongoose'), // ע��Mongoose�����ļ�������server.js�е�һ�����ص������ļ����Ա���Mongoose������ɺ��κ�ģ��������ر����ֱ��ʹ��Mongooseģ�͡�
express = require('./config/express');

var db = mongoose();
var app = express();
app.listen(3000);

module.exports = app;

console.log('Server running at http://localhost:3000/');
```

###1. Mongoose��ģʽ��ģ��
Mongooseʹ��ģʽ�������ĵ��ĸ������ԣ�ÿ�����Զ��������ͺ�Լ�����Ա�����ĵ��Ľṹ��
#####1. ����Userģʽ��ģ��
��app/models�ļ����´���user.server.model.js�ļ����������£�
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
#####2. ע��Userģ��
��config/mongoose.js�ļ��а���user.server.model.js�ļ���
```javascript
...
module.exports = function() {
   var db = mongoose.connect(config.db);
   require('../app/models/user.server.model');
   return db;
};
```


-------------------------------------------------------------------
###2. ģʽ�����������η�����������
#####1. ����Ĭ��ֵ
�磬��UserSchema������һ��created��ʱ���ֶ��������û�ע���ʱ�䡣���ֶν������봴��ʱ�Դ���ʱ����г�ʼ�����档
�༭app/models/user.server.model.js�ļ���
```javascript
   created:{
        type: Date,
        default: Date.now
    },
```
#####2. ʹ��ģʽ���η�
1\. Ԥ�������η�
MongooseԤ������һЩ�򵥵����η�������ַ����͵��ֶ�ʹ��trim���η�ȥ�����˵Ŀո�ʹ��uppercase���η�ת��Ϊ��д��ĸ�ȡ�
ʹ�����£�
```javascript
    username: {
        type: String,
        trim: true, //ȷ���ֶεĿ�ͷ��ĩβ�����пո�
    },
```
2\. �Զ���setter���η�
�Զ���setter���η������ڱ����ĵ�ǰִ�����ݲ���
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
3\. �Զ���getter���η�
�Զ���getter���η������ڽ��ĵ����¼��������ǰ���ĵ����ݽ����޸ġ�
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
��Ϊ��res.json()�ȷ����У��ĵ�ת��ΪJSONĬ�ϲ���ִ��getter���η��Ĳ������������������UserSchema.set()���Ա㱣֤�����������ǿ��ִ��getter���η���
4\. [���������Ϣ�����](http://mongoosejs.com/docs/api.html)

#####3. ������������
��ʱ���ǻ���ҪһЩ��̬������ĵ����ԣ����ֲ���Ҫ�����������洢���ĵ��У���ʱ�����ʹ���������ԡ�
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
#####4. ʹ�������Ż���ѯ������ʹ�ã�
1\. Ψһ����`unique: true`
```javascript
    username: {
        type: String,
        trim: true, 
        unique: true
    },
```
2\. ��������
```javascript
    email: {
        type: String,
        index: true
    },
```

-------------------------------------------------------------------
###3. ʹ��ģ�͵ķ���������ɾ�Ĳ����
#####1. ʹ��save()�������ĵ�
(1) ����һ��Users�����������������������û���صĲ���
��app/controllerĿ¼�´���users.server.controller.js�ļ����������£�
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
(2) ʹ��POST����ע��·��
��app/routesĿ¼�д���users.server.routes.js�ļ����������£�
```javascript
var users = require('../../app/controllers/users.server.controller');
module.exports = function(app) {
   app.route('/users')
   .post(users.create); 
};
```
(3) ����·���ļ�
�޸�config/express.js���£�
```javascript
require('../app/routes/users.server.routes.js')(app);
```
(4) ����
������ʹ��chrome�ġ�����REST��Web����ͻ��ˡ���չ����
ʹ��HTTP��POST��������users�Ļ���·����������������µ�JSON���ݣ�
```json
{
   "firstName": "First",
   "lastName": "Last",
   "email": "user@example.com",
   "username": "username",
   "password": "password"
}
```
#####2. ʹ��find()���Ҷ���ĵ�
(1) ��app/controllers/users.server.controller.js�ļ�������һ��list()������
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
(2) ʹ��GET����ע��·�ɣ�
�޸�app/routes/users.server.routes.js�ļ����£�
```javascript
//other code
app.route('/users')
   .post(users.create)
   .get(users.list);
//other code
```
(3) ������find()�ĸ߼�����
find()���ĸ�������
- Query: MongoDB��ѯ����
- [Fields]:��ѡ������ָ�����ֶ�
- [Options]: ��ѡ����ѯ����ѡ�����
- [Callback]:��ѡ���ص�����
[������ڲ�ѯѡ�����Ϣ�����](http://mongoosejs.com/docs/api.html)

#####3. ʹ��findOne()��ȡ�����ĵ�
findOne()ֻ��ȡ�ض��Ӽ��еĵ�һ���ĵ�
(1) ��app/controllers/users.server.controller.js�ļ�������read()��userByID()������
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
(2) ʹ��GET��app.param()����ע��·��
�޸�app/routes/users.server.routes.js�ļ����£�
```javascript
//other code
app.route('/users/:userId')
   .get(users.read);
app.param('userId', users.userByID);
//other code
```
app.param()������м����������req.user���󣬻����κ� ע��ʱʹ��userId�������м�� ֮ǰִ�У���users.userById()��������users.read()����м��֮ǰִ�С�
(3) ����
ʹ��`$ node server`��������Ӧ�ã�Ȼ�������������http://localhost:3000/users/[id]����ע�����ǰ������ѡȡ���û�_idֵ��[id]�滻һ�¡�
#####4. ���������ĵ�
����������update()��findOneAndUpdate()��findByIdAndUpdate()�ȡ�
����ǰ���Ѿ�������userById()�м�����˴�ʹ��findByIdAndUpdate()��Ϊ���ӡ�
(1) ��app/controllers/users.server.controller.js�ļ�������update()������
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
(2) ʹ��PUT����ע��·��
```javascript
app.route('/users/:userId')
        .get(users.read)
        .put(users.update);
```
(3) ����
���󷽷���PUT
����·����localhost:3000/users/[id]
�����壺{'{"lastName":"Updated"}

#####5. ɾ�������ĵ�
����������remove()��findOneAndRemove()��findByIdAndRemove()��
��������ʹ��findByIdAndRemove()����
(1) ��app/controllers/users.server.controller.js�ļ�������delete()������
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
(2) ʹ��DELETE����ע��·��
```javascript
app.route('/users/:userId')
        .get(users.read)
        .put(users.update)
        .delete(users.delete);
```
(3) ����
���󷽷���DELETE
����·����localhost:3000/users/[id]
#####6. ģ�ͷ����Զ���
1\. �Զ��徲̬����
ģ�;�̬����������ģʽ��statics�����С�
���壨��app/models/user.server.model.js�ļ��ж��壩��
```javascript
UserSchema.statics.findOneByUsername = function (username,callback) {
   this.findOne({ username: new RegExp(username, 'i') }, callback);
};
```
ʹ�ã���app/controllers/users.server.controller.js�ļ���ʹ�ã���
```javascript
User.findOneByUsername('username', function(err, user){
   ...
});
```
2\. �Զ���ʵ������
��ģ���ļ��ж��壺
```javascript
UserSchema.methods.authenticate = function(password) {
   return this.password === password;
};
```
ʹ�ã�
```javascript
var user = new User(req.body);
user.authenticate('password');
```

-------------------------------------------------------------------------------------
###4. ʹ��Ԥ������Զ������֤���������ݽ���У��
While you can validate your data at the logic layer of your application, it is more useful to do it at the model level. Luckily, Mongoose supports both simple predefined validators and more complex custom validators. Validators are defined at the field level of a document and are executed when the document is being saved. If a validation error occurs, the save operation is aborted and the error is passed to the callback.
1\. Ԥ�������֤��
required��֤��������Ӧ��ֵ�Ƿ����
match��֤����ֻ���ܱ�������ʽƥ����Ӧ�ֶε��ĵ����ܱ���
enum��֤�������ֶε�ֵ������޶�
[�������MongooseԤ����֤������Ϣ�����](http://mongoosejs.com/docs/validation.html)
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
2\. �Զ������֤��
Ҫ���Զ�����֤���������ֶε�validate���Լ��ɡ���������һ�����飬�������һ��������һ��������Ϣ��
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
###5. ʹ���м�����ش���ģ�ͷ���
1\. Ԥ�����м��
Ԥ�����м���ڲ���ִ��֮ǰ������For instance, a pre-save middleware will get executed before the saving of the document. This functionality makes pre middleware perfect for more complex validations and default values assignment.�����ӵ���֤����Ĭ��ֵ��ʼ�����ܣ�
ʹ��ģʽ�����`pre()`�������ɶ����봦���м����
```javascript
UserSchema.pre('save', function(next) {
   if (...) {
       next()
   } else {
       next(new Error('An Error Occured'));
   }
});
```
2\. ���ô����м��
���ô����м���ڲ���ִ�����֮�󴥷���For instance, a postsave middleware will get executed after saving the document. This functionality makes post middleware perfect to log your application logic.����־���ܣ�
���ô����м��ʹ��ģʽ�����`post()`�������ж��塣
```javascript
UserSchema.post('save', function(next) {
   if(this.isNew) {
       console.log('A new user was created.');
   } else {
       console.log('A user updated is details.');
   }
});
```
3\. [����Mongoose�м���ĸ�����Ϣ�����](http://mongoosejs.com/docs/middleware.html)

-----------------------------------------
###6. ʹ��Mongoose DBRef
MongoDB�ǲ�֧�����Ӳ�ѯ�ģ�������ʹ��DBRef�ڲ�ͬ���ĵ�֮�佨�����ù�ϵ��Mongooseͨ��ģʽ�е�ObjectID���ͼ�ref������ʵ�ֶ�DBRef��֧�֡�Mongoose��֧���ڲ�ѯʱ�����ĵ���䵽���ĵ��С�
��������ҪΪ���Ĵ���һ��PostSchemaģʽ��ÿ�����Ķ�ͨ��PostSchema��author�ֶ�����Ӧ���ߣ���һ�ֶ���userģ�͵�ʵ�����ɡ�PostSchema�����������:
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
��ע�⣬ref����ָ��ʹ��Userģ�������author�ֶΡ�
��ʹ���У������µĲ���ʱ��������ͨ���������ߴ����ķ�ʽ��ȡUserģ��ʵ��������Userʵ����Ϊ����author�ֶε�ֵ����������:
```javascript
var user = new User();
user.save();

var post = new Post();
post.author = user;
post.save();    
```
post�ĵ��н�����һ�������������ĵ���DBRef������ʱ������˻�ȡ�����ĵ���
DBRef��ֻ�Ǵ��������ĵ���ObjectlD,  Mongoose����Ҫʹ��userʵ�������postʵ�������������ĵ�ʱ��ʹ��populate()��������Խ��û��ĵ���������������δ��룬��ʾ�������find()�Ľ�����ж�author�ֶν������:
```javascript
Post.find().populate('author').exec(function(err, posts) {
   ...
});
```
�������뽫���������post���ϣ������ĵ���author�ֶλ�ʹ�ö�Ӧ��user������䡣
DBRef��MongoDBһ���ܰ��Ĺ��ܣ�Mongoose����һ���ܵ�֧�֣�ʹ��ģ�Ϳ���ͨ����������ʵ�ָ��õ���֯���������������У�Ҳ����DBRef��֧��Ӧ���߼���
[�������DBRef����Ϣ�����](http://mongoosejs.com/docs/populate.html)












###1. MongoDB的设计目标
1. 创建一个既有关系数据库的鲁棒性（健壮性），又能通过分布式扩展快速提升键值存储生产力的数据库。
2. 支持以标准化的JSON输出进行WEB应用开发。
&&
1. 兼具传统数据库特性以及NoSQL存储的高性能
2. 扩展常规键值的存储功能

------------------------------------------------------------
###2. MongoDB的关键特性
1. BSON格式
2. MongoDB即席查询
3. MongoDB索引
索引机制解决了怎样在大量的数据中进行高效查询这一问题。
4. MongoDB副本集 [更多请戳](http://docs.mongodb.org/manual/replication/)
5. MongoDB分片 [更多请戳](http://docs.mongodb.org/manual/sharding/)

------------------------------------------------------
###3. BSON数据结构
BSON（Binary JSON）：将类JSON文档序列化后进行二进制编码。
除了支持JSON的特殊数据类型，还支持其他几种数据类型，比如时间类型Date。
使用_id做主键：_id字段通常是以ObjectId为名的唯一标签符。它要么由应用驱动生成，要门由mongod服务生成。
优点：
使得MongoDB可以高吞吐率地进行读/写操作；
使用_id做主键；
提高了搜索集合的性能；
支持使用复杂查询表达式来匹配对象。

-------------------------------------------------------
###4. 命令行工具的使用
1\. 在本地搭建一个MongoDB环境
注意要配置环境变量
[install-mongodb-on-windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
2\. 启动MongoDB命令行工具
`$ mongo`
此时不指定数据库，会自动连接到默认的test数据库。。
3\. 切换数据库
`> mongo mean`
4\. 连接到指定数据库
`$ mongo 指定的数据库名`
5\. 列出当前连接服务器上的所有数据
`> show dbs`

------------------------------------------------------
###5. 集合与文档
MongoDB集合就是MongoDB文档的列表。集合在插入第一个文档时自动创建。
1\. 创建一个posts集合，并插入第一个博文文档
`> db.posts.insert({"title":"First Post", "user": "bob"})`
2\. 检索集合中的所有文档
`> db.posts.find()`
3\. 查看所有可用的集合
`> show collections`
4\. 删除posts集合
`> db.posts.drop()`

-------------------------------------------------------
###6. 查询语言
CRUD（Create Read Update Delete，增删改查）
####创建新文档
1\. insert()
下面的命令将新文档插入到posts集合中
`> db.posts.insert({"title":"Second Post", "user": "alice"})
2\. update()
当查询条件匹配不到文档时，结合使用upsert标签，便可以插入新文档。
```shell
> db.posts.update({
   "user": "alice"
}, {
   "title": "Second Post",
   "user": "alice"
}, {
   upsert: true
})
```
3\. save()
当传给它的文档没有_id字段，或者_id字段在目前的集合中并没有被使用时，便可创建新文档，如下：
`> db.posts.save({"title":"Second Post", "user": "alice"})`
####读取文档
1\. 查询整个集合中的文档
`> db.posts.find()` 或 `> db.posts.find({})`
2\. 使用等值表达式
检索出所有user值等于alice的文档：
`> db.posts.find({ "user": "alice" })`
3\. 使用查询字符串
要检索posts集合中所有user值为alice或bob的文档，可以使用$in操作符：
`> db.posts.find({ "user": { $in: ["alice", "bob"] } })`
[更多查询操作符的信息请戳](http://docs.mongodb.org/manual/reference/operator/query/#query-selectors)
4\. 创建AND/OR查询
AND操作符：直接将需要检查的属性添加到查询对象即可
`> db.posts.find({ "user": "alice", "commentsCount": { $gt: 10 } })`
OR操作符：需要使用$or操作符：
`> db.posts.find( { $or: [{ "user": "alice" }, { "user": "bob" }] })`
####更新已有文档
1\. update()
语法`db.collection.update(query, update, options)`
用例如下：
```shell
> db.posts.update({
   "user": "alice"
}, {
   $set: {
       "title": "Second Post"
   }
}, {
   multi: true
})
```
上面的例子中，第一个参数是告诉MongoDB找出所有user值为alice的文档，第二个参数是明确更新title字段，第三个参数是更新选项，告知MongoDB更新所有符合条件的文档。
2\. save()
注意传入的文档必须包含有_id字段。save()方法会按_id查找相应的文档，如果找不到则会新建一个。
如：
```shell
> db.posts.save({
   "_id": ObjectId("50691737d386d8fadbd6b01d"),
   "title": "Second Post",
   "user": "alice"
});
```

####删除文档
语法：`db.collection.remove()`
1\. 删除所有文档
`> db.collection.remove()`
要注意remove()方法与drop()方法的区别。前者是删除集合内的所有文档，后者是删除整个集合，包括这个集合的索引。如果要使用不同的索引重建整个集合，推荐使用drop()方法。
2\. 删除多个文档
`> db.posts.remove({ "user": "alice" })`将会删除所有user为alice的文档。
3\. 删除一个文档
`> db.posts.remove({ "user": "alice" }, true)`会删除第一个符合条件的文档。
















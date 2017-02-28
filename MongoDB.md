[TOC]
###1. MongoDB�����Ŀ��
1. ����һ�����й�ϵ���ݿ��³���ԣ���׳�ԣ�������ͨ���ֲ�ʽ��չ����������ֵ�洢�����������ݿ⡣
2. ֧���Ա�׼����JSON�������WEBӦ�ÿ�����
&&
1. ��ߴ�ͳ���ݿ������Լ�NoSQL�洢�ĸ�����
2. ��չ�����ֵ�Ĵ洢����

------------------------------------------------------------
###2. MongoDB�Ĺؼ�����
1. BSON��ʽ
2. MongoDB��ϯ��ѯ
3. MongoDB����
�������ƽ���������ڴ����������н��и�Ч��ѯ��һ���⡣
4. MongoDB������ [�������](http://docs.mongodb.org/manual/replication/)
5. MongoDB��Ƭ [�������](http://docs.mongodb.org/manual/sharding/)

------------------------------------------------------
###3. BSON���ݽṹ
BSON��Binary JSON��������JSON�ĵ����л�����ж����Ʊ��롣
����֧��JSON�������������ͣ���֧�����������������ͣ�����ʱ������Date��
ʹ��_id��������_id�ֶ�ͨ������ObjectIdΪ����Ψһ��ǩ������Ҫô��Ӧ���������ɣ�Ҫ����mongod�������ɡ�
�ŵ㣺
ʹ��MongoDB���Ը������ʵؽ��ж�/д������
ʹ��_id��������
������������ϵ����ܣ�
֧��ʹ�ø��Ӳ�ѯ���ʽ��ƥ�����

-------------------------------------------------------
###4. �����й��ߵ�ʹ��
1\. �ڱ��شһ��MongoDB����
ע��Ҫ���û�������
[install-mongodb-on-windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
2\. ����MongoDB�����й���
`$ mongo`
��ʱ��ָ�����ݿ⣬���Զ����ӵ�Ĭ�ϵ�test���ݿ⡣��
3\. �л����ݿ�
`> mongo mean`
4\. ���ӵ�ָ�����ݿ�
`$ mongo ָ�������ݿ���`
5\. �г���ǰ���ӷ������ϵ���������
`> show dbs`

------------------------------------------------------
###5. �������ĵ�
MongoDB���Ͼ���MongoDB�ĵ����б������ڲ����һ���ĵ�ʱ�Զ�������
1\. ����һ��posts���ϣ��������һ�������ĵ�
`> db.posts.insert({"title":"First Post", "user": "bob"})`
2\. ���������е������ĵ�
`> db.posts.find()`
3\. �鿴���п��õļ���
`> show collections`
4\. ɾ��posts����
`> db.posts.drop()`

-------------------------------------------------------
###6. ��ѯ����
CRUD��Create Read Update Delete����ɾ�Ĳ飩
####�������ĵ�
1\. insert()
�����������ĵ����뵽posts������
`> db.posts.insert({"title":"Second Post", "user": "alice"})
2\. update()
����ѯ����ƥ�䲻���ĵ�ʱ�����ʹ��upsert��ǩ������Բ������ĵ���
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
�����������ĵ�û��_id�ֶΣ�����_id�ֶ���Ŀǰ�ļ����в�û�б�ʹ��ʱ����ɴ������ĵ������£�
`> db.posts.save({"title":"Second Post", "user": "alice"})`
####��ȡ�ĵ�
1\. ��ѯ���������е��ĵ�
`> db.posts.find()` �� `> db.posts.find({})`
2\. ʹ�õ�ֵ���ʽ
����������userֵ����alice���ĵ���
`> db.posts.find({ "user": "alice" })`
3\. ʹ�ò�ѯ�ַ���
Ҫ����posts����������userֵΪalice��bob���ĵ�������ʹ��$in��������
`> db.posts.find({ "user": { $in: ["alice", "bob"] } })`
[�����ѯ����������Ϣ���](http://docs.mongodb.org/manual/reference/operator/query/#query-selectors)
4\. ����AND/OR��ѯ
AND��������ֱ�ӽ���Ҫ����������ӵ���ѯ���󼴿�
`> db.posts.find({ "user": "alice", "commentsCount": { $gt: 10 } })`
OR����������Ҫʹ��$or��������
`> db.posts.find( { $or: [{ "user": "alice" }, { "user": "bob" }] })`
####���������ĵ�
1\. update()
�﷨`db.collection.update(query, update, options)`
�������£�
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
����������У���һ�������Ǹ���MongoDB�ҳ�����userֵΪalice���ĵ����ڶ�����������ȷ����title�ֶΣ������������Ǹ���ѡ���֪MongoDB�������з����������ĵ���
2\. save()
ע�⴫����ĵ����������_id�ֶΡ�save()�����ᰴ_id������Ӧ���ĵ�������Ҳ�������½�һ����
�磺
```shell
> db.posts.save({
   "_id": ObjectId("50691737d386d8fadbd6b01d"),
   "title": "Second Post",
   "user": "alice"
});
```

####ɾ���ĵ�
�﷨��`db.collection.remove()`
1\. ɾ�������ĵ�
`> db.collection.remove()`
Ҫע��remove()������drop()����������ǰ����ɾ�������ڵ������ĵ���������ɾ���������ϣ�����������ϵ����������Ҫʹ�ò�ͬ�������ؽ��������ϣ��Ƽ�ʹ��drop()������
2\. ɾ������ĵ�
`> db.posts.remove({ "user": "alice" })`����ɾ������userΪalice���ĵ���
3\. ɾ��һ���ĵ�
`> db.posts.remove({ "user": "alice" }, true)`��ɾ����һ�������������ĵ���
















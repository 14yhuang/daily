Hbase

CDH-5.12  HBase1.2.0

http://blog.itpub.net/30089851/viewspace-2061399/



Phoenix

Apache Phoenix ：4.14.0-HBase-1.2 （Apache HBase）

要装cdh的版本

4.14.0-cdh5.11.2



[hadoop@hadoop000 bin]$ start-hbase.sh

hbase shell



# hbase shell

## 创建表

```
create 'test','cf'

看看表结构

describe 'test'
```



## 插入数据

```
//row1是rowkey
put 'test','1','cf:a','www.ruoze.com'

//再放
put 'test','1','cf:a','www.ruoze.com,jpson'
//只显示最新版本的
scan 'test'
```



## 查询数据

scan全表扫

```
//全表扫描
scan 'test'
```

get指定rowkey查

```
get 'test','1'
```



## 删除数据

```
delete 'test,'1','cf:a'
```



# Phoneix

hbase命令多麻烦呀，用phoneix就可以用sql写了。

还可以解决二级索引问题。

还可以结合spark。



## 如何使用phoneix



1.找到server的jar包

phoenix-4.14.0-cdh5.11.2-server.jar



2.将它放到regionServer所在的hbase lib目录

如果是cdh管理，那么lib目录在

/opt/cloudera/parcels/CDH/lib/hbase/lib/



3.重启hbase



4.将client的jar包复制到客户端（比如DBeaver ）

![1553510457905](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553510457905.png)

jar包就是驱动

![1553510537586](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553510537586.png)





host里填zookeeper地址

![1553516681812](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553516681812.png)







## 基本语法

```
CREATE TABLE ruozedata(
id INTEGER PRIMARY KEY,
name VARCHAR(20)
);

UPSERT INTO ruozedata(id,name)
VALUES(1,'100');

SELECT * FROM ruozedata;
```

mysql和phoenix类型对应关系

```
mysql   phoenix

char/varchar--》varchar

int--》integer

datetime--》timestamp
```





## 注意事项

DBeaver创建的表，在hbase是大写的。








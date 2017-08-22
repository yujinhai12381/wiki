## 环境及版本
```
docker
Centos : CentOS Linux release 7.3.1611 (Core)
Mongodb: 3.2.10
```

## 安装mongodb
1、安装yum源

/etc/yum.repos.d目录创建mongodb-org-3.2.repo
内容为:

```
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc
```
创建完成后
```
yum clean all
yum makecache
```
验证
```
yum info mongodb-org
```

2、安装
```
yum install mongodb-org-3.2.10
```

## Mongo配置
1、在/data/mongodb目录下创建
```
mkdir conf db log
```
2、在conf目录下创建配置文件mongodb.conf

replSet=test-rep 以副本集模式启动,否则为单机模式
```
dbpath=/data/mongodb/db
logpath=/data/mongodb/log/mongodb.log
port=27017
fork=true
replSet=test-rep
```
如果修改/etc/mongod.conf配置文件可参考
* [Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options/)

3、启动mongodb
```
mongod -f mongodb/conf/mongodb.conf
```
备注
```
我本次采用不同配置文件,本机启动了3个mongodb实例
端口号分别为27017、27018、27019
```

## 副本集配置
```
rs.initiate()
rs.add("172.17.0.2:27017")
rs.add("172.17.0.2:27018")
rs.add("172.17.0.2:27019")
```

仲裁节点添加命令
```
rs.addArb('somehost:30000')
```
仲裁节点的作用：
```
通过实际测试发现，当整个副本集集群中达到50%的节点（包括仲裁节点）不可用的时候，
剩下的节点只能成为secondary节点，整个集群只能读不能写。比如集群中有1个primary节点，
2个secondary节点，加1个arbit节点时：当两个secondary节点挂掉了，
那么剩下的原来的primary节点也只能降级为secondary节点；
当集群中有1个primary节点，1个secondary节点和1个arbit节点，这时即使primary节点挂了，
剩下的secondary节点也会自动成为primary节点。因为仲裁节点不复制数据，
因此利用仲裁节点可以实现最少的机器开销达到两个节点热备的效果。
```
查看副本集状态
```
rs.status() 
```
查看Replica Set的配置信息
```
rs.conf()
```
设置从库可读
```
rs.slaveOk();
```

## 验证
1、primary write-> secondary read 是否一致

2、kill primary  secondary是否切成primary

3、待补充

## 参考文档

* [mongo官方安装文档](https://docs.mongodb.com/v3.2/tutorial/install-mongodb-on-red-hat/)
* [我们的一个已投产项目的高可用数据库实战-mongo副本集的搭建详细过程
](http://www.2cto.com/database/201602/491168.html)
* [MongoDB副本集概述
](http://www.cnblogs.com/magialmoon/p/3251330.html)

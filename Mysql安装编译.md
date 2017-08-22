本文主要参考http://www.rfyy.net/archives/2015.html
以下为自己操作步骤

# 1、环境

```
centos版本：CentOS release 6.8 (Final)  

内核版本：2.6.32-504.el6.x86_64

mysql版本：mysql-5.6.35
```

# 2、下载安装cmake 
mysql5.5以后是通过cmake来编译的

```
yum -y install cmake
```

# 3、安装编译源码所需的工具和库


```
yum install gcc gcc-c++ ncurses-devel perl

```

# 3、安装编译源码所需的工具和库
```
yum install gcc gcc-c++ ncurses-devel perl
```

# 4、创建mysql的数据库存放目录

```
mkdir -p /data/mysql
```

# 5、 创建mysql用户及用户组
```
groupadd -r -g 306 mysql
useradd -r -g mysql -u 306 mysql
```

# 6、编译安装MySQL
```
cd /usr/local/src
wget http://cdn.mysql.com/Downloads/MySQL-5.6/mysql-5.6.35.tar.gz
tar -xf mysql-5.6.35.tar.gz
cd mysql-5.6.35
```

我的编译选项
设置目录为 /usr/local/lnmp/mysql5.6

端口号为6000

```
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/lnmp/mysql5.6 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DSYSCONFDIR=/etc \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DMYSQL_DATADIR=/data/mysql \
-DMYSQL_TCP_PORT=6000 \
-DENABLE_DOWNLOADS=1
```
cmake 选项可参考[官方文档](https://dev.mysql.com/doc/refman/5.5/en/source-configuration-options.html)
```
make && make install
```

# 7、配置MySQL

## 7.1设置权限
```
chown -R mysql:mysql /data/mysql
```

## 7.2初始化配置
进入安装路径
```
cd /usr/local/mysql
```
进入安装路径，执行初始化配置脚本，创建系统自带的数据库和表

```
scripts/mysql_install_db --basedir=/usr/local/lnmp/mysql5.6 --datadir=/data/mysql --user=mysql
```

## 7.3提供配置文件
```
cp ./support-files/my-default.cnf /etc/my.cnf
```

修改配置文件

```
vim /etc/my.cnf 改为如下：
basedir = /usr/local/lnmp/mysql5.6
datadir = /data/mysql
```
## 7.4启动MySQL
添加服务，拷贝服务脚本到init.d目录，并设置开机启动
```
cp support-files/mysql.server /etc/init.d/mysql
chkconfig mysql on
service mysql start --启动MySQL
ss -nat |grep 6000
LISTEN 0 80 :::6000 :::*
```

# 8、设置PATH，要不不能直接调用mysql

修改/etc/profile文件，在文件末尾添加
```
PATH=/usr/local/mysql/bin:$PATH
export PATH
```
关闭文件，运行下面的命令，让配置立即生效
```
source /etc/profile
```
现在，我们可以在终端内直接输入mysql进入，mysql的环境了

# 9、登录
由于使用非默认端口3306，而是6000
```
mysql -u mysql -p -P6000
```

# 遇到问题

## Q1：如果没有mysql库
A:
权限问题 [参考资料](http://blog.csdn.net/liuyifeng1920/article/details/49818851)

##  Q2: 修改Mysql 用户密码
```
>mysql -u root
mysql> use mysql;
mysql> UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
```

## Q3: 远程登录不上
各种登录不上，各种设置权限用户，不管用~ 最后老司机指路 登录端口号没加
登录命令+ P6000 解决



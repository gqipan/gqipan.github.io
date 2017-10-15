---
layout: post
title:  "1、MySQL 的架构介绍"
categories: MySQL
tags: MySQL高级知识
---


* content
{:toc}


### 1、MySQL 的架构介绍

### 1、MySQL 简介

#### 概述
* MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle公司。
* MySQL是一种关联数据库管理系统，将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。
 
* Mysql是开源的，所以你不需要支付额外的费用。
* Mysql支持大型的数据库。可以处理拥有上千万条记录的大型数据库。
* MySQL使用标准的SQL数据语言形式。
* Mysql可以允许于多个系统上，并且支持多种语言。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby和Tcl等。
* Mysql对PHP有很好的支持，PHP是目前最流行的Web开发语言。
* MySQL支持大型数据库，支持5000万条记录的数据仓库，32位系统表文件最大可支持4GB，64位系统支持最大的表文件为8TB。
* Mysql是可以定制的，采用了GPL协议，你可以修改源码来开发自己的Mysql系统。

##### 高级MySQL涉及到知识

* mysql内核
* sql优化攻城狮
* mysql服务器的优化
* 各种参数常量设定
* 查询语句优化
* 主从复制
* 软硬件升级
* 容灾备份
* sql编程
* **完整的mysql优化需要很深的功底，大公司甚至有专门的DBA写上述**

#### 2、MySQL Linux版的安装

* 此次安装的是 MySQL 5.5， 安装环境 CentOS 6.5
* 版本下载地址 [官网下载地址](http://dev.mysql.com/downloads/mysql/)
	* 下载 **MySQL-Client** 和 **MySQL-Server**
	* http://downloads.skysql.com/archives/mysql-5.5/MySQL-server-5.5.16-1.rhel4.i386.rpm
	* http://downloads.skysql.com/archives/mysql-5.5/MySQL-client-5.5.16-1.rhel4.i386.rpm
	* http://downloads.skysql.com/archives/mysql-5.5/MySQL-devel-5.5.16-1.rhel4.i386.rpm
* 检查当前系统是否安装过MySQL
	* 查询命令:  `rpm -qa|grep -i mysql `
	* 删除命令:  `rpm -e --nodeps RPM包全名`
* 安装mysql服务端(**注意提示**)
	* ![设置密码提示](http://ww2.sinaimg.cn/mw690/afac410dgw1fbfs72ir5uj20jf09wabp.jpg)
* 安装mysql客户端
* 查看MySQL安装时创建的mysql用户和mysql组
	* `# cat /etc/passwd | grep mysql`
	* `# cat /etc/group | grep mysql`
* mysql服务的启+停
	* 查看MySQL启停状态:  `# ps -ef | grep mysql`
	* 启停操作:
		* `# /etc/init.d/mysql start`
		* `# /etc/init.d/mysql stop`
		* 或者
		* `#service mysql start`
		* `#service mysql stop`
	* 设置MySQL 自启服务
		* `#chkconfig mysql on`   设置自动启动
		* `# chkconfig --list | grep mysql`   检查是否设置了自动启动
	* 修改配置文件位置
		* 拷贝当前**5.5版本**： `cp /usr/share/mysql/my-huge.cnf /etc/my.cnf`
		*  **5.6版本**  `cp /usr/share/mysql/my-default.cnf /etc/my.cnf`
	*  修改字符集和数据存储路径
		*  查看字符集
			* `show variables like 'character%';` 
			* `show variables like '%char%';`
			* ![字符集](http://ww4.sinaimg.cn/mw690/afac410dgw1fbftd9piuzj20ey0g876s.jpg)
			* 默认的是客户端和服务器都用了latin1，所以会乱码。
		* 修改字符集，修改之前copy 的配置文件。（详细后续代码）
		* MySQL的安装位置
			* 在linux下查看安装目录 `ps -ef|grep mysql`

			| 路径      |     解释 |   备注   |
			| :-------- | --------:| :------: |
			| /var/lib/mysql/ |   mysql数据库文件的存放路径|  /var/lib/mysql/atguigu.cloud.pid|
			| /usr/share/mysql |   配置文件目录|  mysql.server命令及配置文件|
			| /usr/bin     |   相关命令目录|  mysqladmin mysqldump等命令|
			| /etc/init.d/mysql |   启停相关脚本|  |

![MySQL安装位置](http://ww3.sinaimg.cn/large/afac410dgw1fbftsj4yzwj20xm02iwfr.jpg)

```js
[client]
#password = your_password
port = 3306
socket = /var/lib/mysql/mysql.sock

# 这一行需要设置字符集
default-character-set=utf8
 
# The MySQL server
[mysqld]
port = 3306

# 还有这三行
character_set_server=utf8
character_set_client=utf8
collation-server=utf8_general_ci

socket = /var/lib/mysql/mysql.sock
skip-external-locking
key_buffer_size = 384M
max_allowed_packet = 1M
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency = 8
 
[mysql]
no-auto-rehash
# 还有这一行
default-character-set=utf8

```

#### 3、Mysql配置文件

##### 主要配置文件

* 二进制日志log-bin
	* 主从复制
	* ![](http://ww3.sinaimg.cn/large/afac410dgw1fbftxcux5xj20ac01y74g.jpg)
* 错误日志log-error
	* 默认是关闭的,记录严重的警告和错误信息，每次启动和关闭的详细信息等。
* 查询日志log
	* 默认关闭，记录查询的sql语句，如果开启会减低mysql的整体性能，因为记录日志也是需要消耗系统资源的
* 数据文件
	* 两系统
		* windows
			* D:\devSoft\MySQLServer5.5\data目录下可以挑选很多库
		* Linux:  
			* 默认路径 `#cd /var/lib/mysql/`
			* 看看当前系统中的全部库后再进去 `#ls -1F | grep ^d`
	* **frm文件**: 存放表结构
	* **myd文件:** 存放表数据
	* **myi文件:** 存放表索引
* 如何配置
	*  Windows: my.ini文件
	*  Linux:  /etc/my.cnf文件


#### 4、Mysql逻辑架构介绍

##### 总体概览
*   和其它数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上，**插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离**。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。
	* ![](http://ww1.sinaimg.cn/large/afac410dgw1fbfuxrf6q4j20k70dladf.jpg)
	* **1、连接层**
		* 最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。
	* **2、服务层**
		* 第二层架构主要完成大多少的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存。如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。
	* **3、引擎层**
		*  存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。后面介绍MyISAM和InnoDB
	* **4、存储层**
		* 数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。

##### 查询说明
* 首先，mysql的查询流程大致是：
	* mysql客户端通过协议与mysql服务器建连接，发送查询语句，先检查查询缓存，如果命中，直接返回结果，否则进行语句解析
	* 有一系列预处理，比如检查语句是否写正确了，然后是查询优化（比如是否使用索引扫描，如果是一个不可能的条件，则提前终止），生成查询计划，然后查询引擎启动，开始执行查询，从底层存储引擎调用API获取数据，最后返回给客户端。怎么存数据、怎么取数据，都与存储引擎有关。
	* 然后，mysql默认使用的BTREE索引，并且一个大方向是，无论怎么折腾sql，至少在目前来说，mysql最多只用到表中的一个索引。

#### 5、Mysql存储引擎

* 查看命令
	* 查看当前的MySQL 提供什么存储引擎
		* `mysql> show engines;`
	* 看你的 MySQL 当前默认的存储引擎:
		* `show variables like '%storage_engine%';`
	* ![默认的存储引擎](http://ww2.sinaimg.cn/large/afac410dgw1fbfvscn9i1j20bb04rwes.jpg)
* **`MyISAM`**和**`InnoDB`**
	* ![两种引擎对比](http://ww3.sinaimg.cn/large/afac410dgw1fbfvug22pjj20rj0840ub.jpg)
* 阿里巴巴、淘宝用哪个
	* ![](http://ww3.sinaimg.cn/large/afac410dgw1fbfvwphtopj20ma03udgo.jpg)
	*  Percona 为 MySQL 数据库服务器进行了改进，在功能和性能上较 MySQL 有着很显著的提升。该版本提升了在高负载情况下的 InnoDB 的性能、为 DBA 提供一些非常有用的性能诊断工具；另外有更多的参数和命令来控制服务器行为。
	* 该公司新建了一款存储引擎叫**`xtradb`**完全可以替代**`innodb`**,并且在性能和并发上做得更好,
	* 阿里巴巴大部分mysql数据库其实使用的percona的原型加以修改。

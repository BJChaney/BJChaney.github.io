---
layout: post
title:  "Ubuntu MySQL 小记"
date:   2016-07-31 23:24:10 +0800
categories: jekyll update
---
#### 最新在公司转岗了后端开发，在家也打算折腾一下，整理一下折腾Mysql中遇到的一些问题

### 1.安装
   <br>安装服务器：

    root@ubuntu:/# apt-get install mysql-server-5.5
  安装客户端：
  
    root@ubuntu:/# apt-get install mysql-client-core-5.5

中途遇到依赖包无法下载的问题，更换一下软件源就ok 度娘 [ubuntu更新源](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd=ubuntu%E6%9B%B4%E6%96%B0%E6%BA%90&rsv_pq=9b7c5b740040b953&rsv_t=8137zLkahzPvnNNDMNw1AaLhd%2Bu0OFa8xkNvi%2BJ3JjfEa099Zkuj6GHpPDc&rqlang=cn&rsv_enter=1&rsv_sug3=2&rsv_n=2)


***ubuntu下mysql安装目录:***

| url |  content  |
|-|-|
| /usr/bin | 客户端和mysql_install_db |
| /var/lib/mysql | 数据库和日志文件 |
| /var/run/mysqld | 服务器 |
| /etc/mysql | 配置文件 my.cnf |
| /usr/share/mysql | 字符集，基准程序和错误消息 |
| /etc/init.d/mysql | 启动mysql服务器 |


### 2.常用操作
<br>
***mysql服务操作***


    // /etc/init.d/mysql + 命令   start|stop|restart|reload|force-reload|status
    /etc/init.d/mysql start;

***mysql登陆***

    mysql -u 用户名 -p

***查看编码***

    mysql> SHOW VARIABLES LIKE 'character_set_%';
    +--------------------------+----------------------------+
    | Variable_name            | Value                      |
    +--------------------------+----------------------------+
    | character_set_client     | utf8                       |
    | character_set_connection | utf8                       |
    | character_set_database   | latin1                     |
    | character_set_filesystem | binary                     |
    | character_set_results    | utf8                       |
    | character_set_server     | latin1                     |
    | character_set_system     | utf8                       |
    | character_sets_dir       | /usr/share/mysql/charsets/ |
    +--------------------------+----------------------------+
    8 rows in set (0.00 sec)

  ***查看字符集***

    mysql> SHOW VARIABLES LIKE 'collation_%';

    +----------------------+-------------------+
    | Variable_name        | Value             |
    +----------------------+-------------------+
    | collation_connection | utf8_general_ci   |
    | collation_database   | latin1_swedish_ci |
    | collation_server     | latin1_swedish_ci |
    +----------------------+-------------------+
    3 rows in set (0.00 sec)

***修改配置文件编码***

    # client 下添加 default-character-set=utf8
    [client]
    port		= 3306
    socket		= /var/run/mysqld/mysqld.sock
    default-character-set=utf8

    # mysqld 下添加 character-set-server=utf8 （可添加  init_connect='SET NAMES utf8' 设置数据库链接时的编码）
    [mysqld]
    user		= mysql
    pid-file	= /var/run/mysqld/mysqld.pid
    socket		= /var/run/mysqld/mysqld.sock
    port		= 3306
    basedir		= /usr
    datadir		= /var/lib/mysql
    tmpdir		= /tmp
    lc-messages-dir	= /usr/share/mysql
    skip-external-locking
    character-set-server=utf8
    init_connect='SET NAMES utf8'

**配置优化**  [my.cnf参数配置优化详解](http://www.ha97.com/4110.html)


***查看数据库或表的创建信息（查编码）***
     
    show create database `database_name`;
    
    show create table `table_name`;
    

***查看表中所有字段的编码***

    show full columns from `table_name`;

***更改数据库或表的编码和字符集***

    //更改后需重启服务
    ALTER DATABASE `database_name` （ALTER TABLE `table_name`） DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci  

*** 修改表中字段的字符集***
    
     ALTER TABLE `table_name` modify `column_name` 字段类型 CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL;
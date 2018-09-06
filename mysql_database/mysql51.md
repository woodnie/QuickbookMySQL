## MySQL5.1 {#mysql5-1}

第一个 MySQL Community Server，这个不要钱！

第二个 MySQL Enterprise 这个要掏钱，不过可以打电话咨询问题，也就是电话技术支持。

第三个 MySQL Cluster，这个单独是没法用的，要在1或2的基础上用。当然用来平衡多台数据库的。

第四个 MySQL Workbench，这是个好东西，用来设计数据库的。erwin知道吗？他就是这个作用。

MySQL-3.23.39-1.i386.rpm ：包含运行MySQL服务器所需的全部文件，包括客户机程序

MySQL-3.23.39-1.src.rpm：包含MySQL的所有源码

MySQL-bench-3.23.39-1.i386.rpm：包含用于测试MySQL性能的程序， 这些测试程序需要主发行档和Perl

MySQL-client-3.23.39-1.i386.rpm：只包含客户机程序

MySQL-devel-3.23.39-1.i386.rpm&quot;：包含编译客户机程序所需要的库和头文件

MySQL-shared-3.23.39-1.i386.rpm：包含用于客户机程序的共享库

MySQL GUI Tools一个可视化界面的MySQL数据库管理控制台，提供了四个非常好用的图形化应用程序，方便数据库管理和数据查询。这些图形化管理工具可以大大提 高数据库管理、备份、迁移和查询效率，即使没有丰富的SQL语言基础的用户也可以应用自如。它们分别是：

MySQL Migration Toolkit：数据库迁移

MySQL Administrator：MySQL管理器

MySQL Query Browser：用于数据查询的图形化客户端

MySQL Workbench：DB Design工具

For information about MySQL products and services, visit:

  [http://www.mysql.com/](http://www.mysql.com/)

For developer information, including the MySQL Reference Manual, visit:

  [http://dev.mysql.com/](http://dev.mysql.com/)

To buy MySQL Network Support, training, or other products, visit:

  [https://shop.mysql.com/](https://shop.mysql.com/)

安装mysql:

[root@node ~]# yum list mysql*

mysql.x86_64                 #client

mysql-server.x86_64     #server

mysql-devel.x86_64       #devel

mysql-test.x86_64

mysql-bench.x86_64

mysql-connector-odbc.x86_64

MySQL-python.x86_64

#mysql  不区分大小写

#mysql  command以 ；为分割符/结束符

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

提示符

含义

mysql&gt;

准备好接受新命令。

-&gt;

等待多行命令的下一行。

&amp;apos;&gt;

等待下一行，等待以单引号(&quot;&amp;apos;&quot;)开始的字符串的结束。

&quot;&gt;

等待下一行，等待以双引号(&quot;&quot;&quot;)开始的字符串的结束。

`&gt;

等待下一行，等待以反斜点(&amp;apos;`&amp;apos;)开始的识别符的结束。

/*&gt;

等待下一行，等待以/*开始的注释的结束。

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

1.连接服务器：

[root@node ~]# mysql                      #用户以匿名用户(如果被允许的话)连接到本地主机上运行的服务器.

[root@node ~]# mysql -h host -u user -p   #实名登录

mysql -h 192.168.1.100 -u root -p

[root@node ~]# mysql --skip-reconnect     #--skip-reconnect 禁用mysql自动连接

2.输入查询:

mysql&gt; select version();      #查询mysql版本

mysql&gt; select pi();           #查询系统定义的pi值

mysql&gt; select now();          #查询  

mysql&gt; select current_date;   #查询当前日期

mysql&gt; select current_time;   #查询当期时间

mysql&gt; select current_user;

mysql&gt; select WEEK(&amp;apos;2011-08-01&amp;apos;); #查询2011.08.01是一年中的第几周

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

mysql&gt; select       #mysql接受自由格式的输入：它收集输入行但直到看见分号才执行

   -&gt; user()            #用\G而不用分号结束查询可以垂直显示查询

   -&gt; ,

   -&gt; now();

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

mysql&gt; select       # \c取消输入命令      

   -&gt; user()

   -&gt; \c

mysql&gt;

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

mysql&gt; SELECT * FROM my_table WHERE name = &amp;apos;Smith AND age &lt; 30;

        &amp;apos;&gt; &amp;apos;\c             # \c取消输入命令,此命令会因缺少右 &amp;apos;  而引起长时间无结果的错误查询  

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

3.创建并使用数据库

3.1

mysql&gt; show databases;

+--------------------+

| Database           |

+--------------------+

| information_schema |

| menagerie          |

| mysql              |

| test               |

+--------------------+

mysql&gt; user test

mysql&gt; show tables ;

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

3.2创建menagerie实例数据库

mysql&gt; create database menagerie;  #创建menagerie实例数据库

mysql&gt; use menagerie

mysql&gt; show tables ;    #此时为数据库空

Empty set (0.00 sec)

3.3创建数据表：

3.3.1创建pet实例数据表

mysql&gt; CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);  

mysql&gt; show tables;

+---------------------+

| Tables_in_menagerie |

+---------------------+

| pet                 |

+---------------------+

mysql&gt; DESCRIBE pet;          #描述数据表的结构

+---------+-------------+------+-----+---------+-------+

| Field   | Type        | Null | Key | Default | Extra |

+---------+-------------+------+-----+---------+-------+

| name    | varchar(20) | YES  |     | NULL    |       |

| owner   | varchar(20) | YES  |     | NULL    |       |

| species | varchar(20) | YES  |     | NULL    |       |

| sex     | char(1)     | YES  |     | NULL    |       |

| birth   | date        | YES  |     | NULL    |       |

| death   | date        | YES  |     | NULL    |       |

+---------+-------------+------+-----+---------+-------+

3.3.2将数据装入表中

1\. 使用LOAD DATA INFILE &amp;apos;filename&amp;apos;命令

2\. 使用mysqlimport实用程序

3.mysql -u root -p  adidas_11120_dev  &lt;var/www/adidas/database/backup.03-30-2012.sql

#linux中以定位符(Tab)为分割符，建立pet.tab文件

+++++++++++++++++++++++++++++++++++++++++++

| **name** | **owner** | **species** | **sex** | **birth** | **death** |
| --- | --- | --- | --- | --- | --- |
| Fluffy | Harold | cat | f | 1993-02-04 | ~ |
| Claws | Gwen | cat | m | 1994-03-17 | ~ |
| Buffy | Harold | dog | f | 1989-05-13 | ~ |
| Fang | Benny | dog | m | 1990-08-27 | ~ |
| Bowser | Diane | dog | m | 1979-08-31 | 1995-07-29 |
| Chirpy | Gwen | bird | f | 1998-09-11 | ~ |
| Whistler | Gwen | bird | \N | 1997-12-09 | ~ |
| Slim | Benny | snake | m | 1996-04-29 | ~ |

++++++++++++++++++++++++++++++++++++++++++++

3.3.2.1批量增加

mysql&gt; load data local infile &amp;apos;/root/pet.tab&amp;apos; into table pet；

mysql&gt; load data local infile &amp;apos;/root/pet.tab&amp;apos; into table pet lines terminated by &amp;apos;\r\n&amp;apos;;  #如果用Windows中的编辑器(使用\r\n做为行的结束符)创建文件,应使用：

3.3.2.2逐次增加

mysql&gt; insert into pet values (&amp;apos;Puffball&amp;apos;,&amp;apos;Diane&amp;apos;,&amp;apos;hamster&amp;apos;,&amp;apos;f&amp;apos;,&amp;apos;1999-03-30&amp;apos;,NULL);    #一次增加一个新记录

mysql -u root -p 数据库名 &lt;file.sql

4\. 选择并修改数据

4.1选择所有数据

mysql&gt; select * from pet;

+----------+--------+---------+------+------------+------------+

| name     | owner  | species | sex  | birth      | death      |

+----------+--------+---------+------+------------+------------+

| Fluffy   | Harold | cat     | f    | 1993-02-04 | NULL       |

| Claws    | Gwen   | cat     | m    | 1994-03-17 | NULL       |

| Buffy    | Harold | dog     | f    | 1989-05-13 | NULL       |

| Fang     | Benny  | dog     | m    | NULL       | 1990-08-27 |

| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |

| Chirpy   | Gwen   | bird    | f    | 1998-09-11 | NULL       |

| Whistler | Gwen   | bird    | NULL | 1997-12-09 | NULL       |

| Slim     | Benny  | snake   | m    | 1996-04-29 | NULL       |

+----------+--------+---------+------+------------+------------+

4.2修改数据

mysql&gt; DELETE FROM pet;

mysql&gt; load data local infile &amp;apos;/root/pet.tab&amp;apos; into table pet;

mysql&gt; UPDATE pet SET birth = &amp;apos;1989-08-31&amp;apos; WHERE name = &amp;apos;Bowser&amp;apos;; &lt; /FONT &gt;

5.获得数据库和表的信息

mysql&gt; select database();  #显示当前选择了哪个数据库

mysql&gt; SHOW TABLES;        #显示当前的数据库包含什么表

mysql&gt; DESCRIBE pet;       #显示一个表的结构

6.数据导出

1\. 使用select into outfile &amp;apos;filename&amp;apos;语句

2\. 使用mysqldump实用程序

6.1.mysqldump：数据库备份程序

有3种方式来调用mysqldump：

shell&gt; mysqldump [options] db_name [tables]

shell&gt; mysqldump [options] ---database DB1 [DB2 DB3...]

shell&gt; mysqldump [options] --all--database

--opt

该选项是速记；等同于指定 --add-drop-tables--add-locking --create-option --disable-keys--extended-insert --lock-tables --quick --set-charset。它可以给出很快的转储操作并产生一个可以很快装入MySQL服务器的转储文件。该选项默认开启，但可以用--skip-opt禁用。要想只禁用确信用-opt启用的选项，使用--skip形式；例如，--skip-add-drop-tables或--skip-quick。

--extended-insert，-e

使用包括几个VALUES列表的多行INSERT语法。这样使转储文件更小，重载文件时可以加速插入。

--compress，-C

压缩在客户端和服务器之间发送的所有信息（如果二者均支持压缩）。

mysqldump最常用于备份一个整个的数据库：

shell&gt; mysqldump --opt db_name &gt; backup-file.sql

你可以这样将转储文件读回到服务器：

shell&gt; mysql     db_name   &lt;   backup-file.sql

或者为：

shell&gt; mysql  -e  &quot;source    ../backup-file.sql &quot;   db_name

mysqldump也可用于从一个MySQL服务器向另一个服务器复制数据时装载数据库：

shell&gt; mysqldump --opt    db_name   |  mysql --host=remote_host  -C  db_name

可以用一个命令转储几个数据库：

shell&gt; mysqldump ---database  db_name1  [   db_name2 ... ]  &gt;  my_databases.sql

如果你想要转储所有数据库，使用--all--database选项：

shell&gt; mysqldump --all-databases &gt; all_databases.sql

mysqlhotcopy只用于备份MyISAM。它运行在Unix和NetWare中

mysql命令行工具：

4.mysqlaccess：用于检查访问权限的客户端

[root@node ~]# mysqlaccess   root  db_name  node

5.mysqladmin：用于管理MySQL服务器的客户端

6.mysqlbinlog：用于处理二进制日志文件的实用工具

7.mysqlcheck：表维护和维修程序

mysqlcheck客户端可以检查和修复MyISAM表。它还可以优化和分析表

有3种方式来调用mysqlcheck：

shell&gt; mysqlcheck [options] db_name [tables]

shell&gt; mysqlcheck [options] ---database DB1 [DB2 DB3...]

shell&gt; mysqlcheck [options] --all--database

8\. mysqldump：数据库备份程序

9\. mysqlhotcopy：数据库备份程序

mysqlhotcopy只用于备份MyISAM。它运行在Unix和NetWare中。

10\. mysqlimport：数据导入程序

mysqlimport的大多数选项直接对应LOAD DATA INFILE子句

12\. myisamlog：显示MyISAM日志文件内容

11\. mysqlshow－显示数据库、表和列信息

MySQL访问权限系统

[http://blog.chinaunix.net/space.php?uid=23772067&amp;do=blog&amp;id=2386073](http://blog.chinaunix.net/space.php?uid=23772067&do=blog&id=2386073)

GRANT和REVOKE语句间接来操作 授权表的内容，设置账户并控制个人的权限

shell&gt; mysqladmin -u user_name -h host_name password &quot;newpwd&quot;    #使用mysqladmin创建新的登录用户

mysql&gt; GRANT ALL PRIVILEGES ON db.* TO userdb@&amp;apos;192.168.61.100&amp;apos; IDENTIFIED BY &amp;apos;mysql&amp;apos; WITH GRANT OPTION;  

#授权给用户userdb可以从192.168.61.100登录mysql ;

mysql&gt; FLUSH PRIVILEGES;

你可以用几种方法为用户账户指定密码。以下介绍了三种方法：

1.使用SET PASSWORD语句

2.使用mysqladmin命令行客户端程序

3.使用UPDATE语句

1.使用SET PASSWORD指定密码，用root连接服务器并执行两个SET PASSWORD语句。一定要使用PASSWORD()函数来加密密码。

shell&gt; mysql -u root

mysql&gt; SET PASSWORD FOR &amp;apos;root&amp;apos;@&amp;apos;localhost&amp;apos; = PASSWORD(&amp;apos;newpwd&amp;apos;);

mysql&gt; SET PASSWORD FOR &amp;apos;root&amp;apos;@&amp;apos;host_name&amp;apos; = PASSWORD(&amp;apos;newpwd&amp;apos;);

用服务器主机名替换第二个SET PASSWORD语句中的host_name。这是你指定匿名账户密码的主机名。

2.使用mysqladmin为root账户指定密码，执行下面的命令：

 shell&gt; mysqladmin -u root password &quot;newpwd&quot;

 shell&gt; mysqladmin -u root -h host_name password &quot;newpwd&quot;

上述命令适用于Windows和Unix。用服务器主机名替换第二个命令中的host_name。不一定需要将密码用双引号引起来，但是你如果密码中包含空格或专用于命令解释的其它字符，则需要用双引号引起来。

3.使用UPDATE直接修改user表。下面的UPDATE语句可以同时为两个root账户指定密码：

shell&gt; mysql -u root

mysql&gt; UPDATE mysql.user SET Password = PASSWORD(&amp;apos;newpwd&amp;apos;)

   -&gt;     WHERE User = &amp;apos;root&amp;apos;;

mysql&gt; FLUSH PRIVILEGES;

UPDATE语句适用于Windows和Unix。

设置完密码后，当你连接服务器时你必须提供相应密码。例如，如果你想要用mysqladmin 关闭服务器，可以使用下面的命令：

shell&gt; mysqladmin -u root -p shutdown

Enter password: (enter root password here)

删除匿名账户，操作方法是：

shell&gt; mysql -u root

mysql&gt; DELETE FROM mysql.user WHERE User = &amp;apos;&amp;apos;;

mysql&gt; FLUSH PRIVILEGES;

+++++++++++

mysql 修改密码：

[root@secview ~]# mysqladmin -u root -p password newpassword

[root@secview ~]# mysqladmin -u root -p password mzwxl

Enter password: (oldpassword，oldpassword为空时直接回车)

[root@secview ~]#  

+++++

不登陆数据库执行mysql命令小结

1.通过echo实现（这个比较常见）

echo &quot;show databases;&quot; | mysql -uroot -p&amp;apos;oldboy&amp;apos; -S /data/3308/mysql.sock

提示：此法适合单行字符串比较少的情况。

2.通过cat实现（此法用的不多）

cat |mysql -uroot -p&amp;apos;oldboy&amp;apos; -S /data/3308/mysql.sock &lt;&lt; EOF

show databases;

EOF

提示：此法适合多行字符串比较多的时候。

3.通过mysql -e参数实现

mysql -u root -p&amp;apos;oldboy&amp;apos; -S /data1/3307/mysql.sock -e &quot;show databases;&quot;

特殊生产场景应用：

例一：mysql自动批量制作主从同步需要的语句。

cat |mysql -uroot -p&amp;apos;oldboy&amp;apos; -S /data/3308/mysql.sock&lt;&lt; EOF

       CHANGE MASTER TO  

MASTER_HOST=&amp;apos;10.0.0.16&amp;apos;,

MASTER_PORT=3306,

MASTER_USER=&amp;apos;oldboyrep&amp;apos;,

MASTER_PASSWORD=&amp;apos;oldboyrep&amp;apos;,

MASTER_LOG_FILE=&amp;apos;mysql-bin.000025&amp;apos;

MASTER_LOG_POS=4269;

EOF

提示：大家多注意整个语句的写法，而不是cat中的内容。

例二：mysql线程中，&quot;大海捞针&quot;

  平时登陆数据库show processlist;,发现结果经常超长，找自己要看的的比较困难，而且，

SQL显示不全。如果直接执行show full processlist那更是瞬间滚了N屏。找到有问题的

SQL语句非常困难。

现在推荐如下语句。

mysql -u root -p&amp;apos;oldboy&amp;apos; -S /data1/3307/mysql.sock -e &quot;show full processlist;&quot;|grep -v Sleep

过滤当前执行的SQL语句完整内容，这条命令很有用。不知道你能否体会到。后面还可以加iconv等对中文转码，根据需求过滤想要的内容，此命令屡试不爽啊。
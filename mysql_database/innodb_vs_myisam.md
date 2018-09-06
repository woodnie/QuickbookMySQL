## InnoDB VS MyISAM {#innodb-vs-myisam}

mysql 数据据存储引擎InnoDB和MyISAM的优势及区别

发布日期：2010-09-14 13:15来源：作者：

　　MyISAM:这个是默认类型,它是基于传统的ISAM类型,ISAM是Indexed Sequential Access Method (有索引的顺序访问方法) 的缩写,它是存储记录和文件的标准方法.与其他存储引擎比较,MyISAM具有检查和修复表格的大多数工具. MyISAM表格可以被压缩,而且它们支持全文搜索.它们不是事务安全的,而且也不支持外键。如果事物回滚将造成不完全回滚，不具有原子性。如果执行大量的SELECT，MyISAM是更好的选择。

　　InnoDB:这种类型是事务安全的.它与BDB类型具有相同的特性,它们还支持外键.InnoDB表格速度很快.具有比BDB还丰富的特性, 因此如果需要一个事务安全的存储引擎,建议使用它.如果你的数据执行大量的INSERT或UPDATE,出于性能方面的考虑，应该使用InnoDB表,

　　对于支持事物的InnoDB类型的标，影响速度的主要原因是AUTOCOMMIT默认设置是打开的，而且程序没有显式调用BEGIN 开始事务，导致每插入一条都自动Commit，严重影响了速度。可以在执行sql前调用begin，多条sql形成一个事物(即使autocommit打开也可以)，将大大提高性能。

　　===============================================================

　　InnoDB和MyISAM是在使用MySQL最常用的两个表类型，各有优缺点，视具体应用而定。下面是已知的两者之间的差别，仅供参考。

　　innodb

　　InnoDB 给 MySQL 提供了具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。InnoDB 提供了行锁(locking on row level)，提供与 Oracle 类型一致的不加锁读取(non-locking read in SELECTs)。这些特性均提高了多用户并发操作的性能表现。在InnoDB表中不需要扩大锁定(lock escalation)，因为 InnoDB 的列锁定(row level locks)适宜非常小的空间。InnoDB 是 MySQL 上第一个提供外键约束(FOREIGN KEY constraints)的表引擎。

　　InnoDB 的设计目标是处理大容量数据库系统，它的 CPU 利用率是其它基于磁盘的关系数据库引擎所不能比的。在技术上，InnoDB 是一套放在 MySQL 后台的完整数据库系统，InnoDB 在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。 InnoDB 把数据和索引存放在表空间里，可能包含多个文件，这与其它的不一样，举例来说，在 MyISAM 中，表被存放在单独的文件中。InnoDB 表的大小只受限于操作系统的文件大小，一般为 2 GB。

　　InnoDB所有的表都保存在同一个数据文件 ibdata1 中(也可能是多个文件，或者是独立的表空间文件),相对来说比较不好备份，免费的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump。

　　MyISAM

　　MyISAM 是MySQL缺省存贮引擎 .

　　每张MyISAM 表被存放在三个文件 。frm 文件存放表格定义。 数据文件是MYD (MYData) 。 索引文件是MYI (MYIndex) 引伸。

　　因为MyISAM相对简单所以在效率上要优于InnoDB..小型应用使用MyISAM是不错的选择.

　　MyISAM表是保存成文件的形式,在跨平台的数据转移中使用MyISAM存储会省去不少的麻烦

　　以下是一些细节和具体实现的差别：

　　1.InnoDB不支持FULLTEXT类型的索引。

　　2.InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。

　　3.对于AUTO_INCREMENT类型的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。

　　4.DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。

　　5.LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性(例如外键)的表不适用。

　　另外，InnoDB表的行锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表，例如update table set num=1 where name like &quot;%aaa%&quot;

　　任何一种表都不是万能的，只用恰当的针对业务类型来选择合适的表类型，才能最大的发挥MySQL的性能优势。

　　===============================================================

　　以下是InnoDB和MyISAM的一些联系和区别!

　　1\. 4.0以上mysqld都支持事务，包括非max版本。3.23的需要max版本mysqld才能支持事务。

　　2\. 创建表时如果不指定type则默认为myisam，不支持事务。

　　可以用 show create table tablename 命令看表的类型。

　　2.1 对不支持事务的表做start/commit操作没有任何效果，在执行commit前已经提交，测试：

　　执行一个msyql：

　　use test;

　　drop table if exists tn;

　　create table tn (a varchar(10)) type=myisam;

　　drop table if exists ty;

　　create table ty (a varchar(10)) type=innodb;

　　begin;

　　insert into tn values(&amp;apos;a&amp;apos;);

　　insert into ty values(&amp;apos;a&amp;apos;);

　　select * from tn;

　　select * from ty;

　　都能看到一条记录

　　执行另一个mysql：

　　use test;

　　select * from tn;

　　select * from ty;

　　只有tn能看到一条记录

　　然后在另一边

　　commit;

　　才都能看到记录。

　　3\. 可以执行以下命令来切换非事务表到事务(数据不会丢失)，innodb表比myisam表更安全：

　　alter table tablename type=innodb;

　　3.1 innodb表不能用repair table命令和myisamchk -r table_name

　　但可以用check table，以及mysqlcheck [OPTIONS] database [tables]

　　4\. 启动mysql数据库的命令行中添加了以下参数可以使新发布的mysql数据表都默认为使用事务(

　　只影响到create语句。)

　　--default-table-type=InnoDB

　　测试命令：

　　use test;

　　drop table if exists tn;

　　create table tn (a varchar(10));

　　show create table tn;

　　5\. 临时改变默认表类型可以用：

　　set table_type=InnoDB;

　　show variables like &amp;apos;table_type&amp;apos;;

　　或：

　　c:\mysql\bin\mysqld-max-nt --standalone --default-table-type=InnoDB
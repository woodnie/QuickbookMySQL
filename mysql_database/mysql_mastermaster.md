## mysql master/master {#mysql-master-master}

[http://www.linuxany.com/archives/1376.html#more-1376](http://www.linuxany.com/archives/1376.html)

mysql+mmm+proxy 实现数据库读写分离及高可用HA

Master-Slave的数据库机构解决了很多问题，特别是read/write比较高的web2.0应用：

1、写操作全部在Master结点执行，并由Slave数据库结点定时(默认60s)读取Master的bin-log

2、将众多的用户读请求分散到更多的数据库节点，从而减轻了单点的压力

这是对Replication的最基本陈述，这种 模式的在系统Scale-out方案中很有引力(如 有必要，数据可以先进行Sharding，再使用replication)。

它的缺点是：

1、Slave实时性的保障，对于实时性很高的场合 可能需要做一些处理

2、高可用性问题，Master就是那个致命点(SPOF:Single point of failure)

本文主要讨论的是如何解决第2个缺点。

DB的设计对大规模、高负载的系统是极其重要的。高可用性(High availability)在重要的系统(critical System)是需要架构师事先 考虑的。存在SPOF:Single point of failure的设计在 重要系统中是危险的。

Master-Master Replication

1、使用两个MySQL数据库db01,db02，互为Master和Slave，即：

一边db01作为db02的master，一旦有数据写向db01时，db02定时从db01更新

另一边db02也作为db01的master，一旦有数据写向db02时，db01也定时从db02获得更新

(这不会导致循环，MySQL Slave默认不会 记录Master同步过来的变化)

2、但从AppServer的角度来说，同时只有一 个结点db01扮演Master，另外一个 结点db02扮演Slave，不能同时两个 结点扮演Master。即AppSever总 是把write操作分配某个数据库(db01)， 除非db01 failed，被切换。

3、如果扮演Slave的数据库结点db02 Failed了：

a)此时appServer要能够把所有的read,write分配给db01，read操作不再指向db02

b)一旦db02恢复过来后，继续充当Slave角色，并告诉AppServer可以将read分配给它了

4、如果扮演Master的数据库结点db01 Failed了

a)此时appServer要能够把所有的写操作从db01切换分配给db02，也就是切换Master由db02充当

b)db01恢复过来后，充当Slave的角色，Master由db02继续扮演

难点：

3、4要如何自动进行？

Master-Master with n Slaves Replication

这比上一个还要复杂，即：

当一个Master Fail时，所有的Slave不再从原来失败的那个Master(db01)获 取更新日志，而应该&quot;自动&quot;切换到最新充当Master角色的数据库db02。

MMM，a greate project!

MMM的基本信息请参考它的网站(见后&quot;参考资料&quot;)

MMM有3个重要的器件:

1、mmmd_mon - monitoring script which does all monitoring work and makes all decisions about roles moving and so on.

2、mmmd_agent - remote servers management agent script, whichprovides monitoring node with simple set of remote services to makeservers management easier, more flexible abd highly portable.

3、mmm_control - simple script dedicated to management of the mmmd_mon processes by commands.

每一个MySQL服务器器结点需要运行mmmd_agent，同时在另外的一个机器上(可以 是独立的一台机器，也可以是和AppServer共享同一个服务器)运行mmmd_mon。形成1 * mmmd_mon + n * mmmd_agent的部署架构。

MMM利用了虚拟IP的技术：1个网卡可以同时使用多个IP。

(所以使用MMM时，需要2*n+1个IP，n为mysql数据库结点个数，包括master,slave)

当有数据库结点fail时，mmmd_mon检测不到mmmd_agent的心跳或 者对应的MySQL服务器的状态，mmmd_mon将 进行决定，并下指令给某个正常的数据库结点的mmmd_agent，使得该mmmd_agent&quot;篡位&quot;使用(注)刚才fail的那个结点的虚拟IP，使得虚拟IP实际从指向fail的那个机器自动转为此时的这个正 常机器。

注：据Qieqie猜测是将获得的虚拟IP设置给网卡，也只能这样了，改天测试验证一下。

repeat: MMM对MySQL Master-Slave Replication绝对是一个很有益的补充!

整体架构的原理：

Webclient 数据请求至proxy-proxy进行读写分发－转至mmm机制－在检测存活 的机器进行读与写操作。在此之前这些机器与为master/slave.

本文测试环境如下：

主机名 IP Port App 目录 备注

Node1 192.168.1.2 3306 mysql /var/lib/mysql 数据库服务器1

Node2 192.168.1.3 3306 mysql /var/lib/mysql 数据库服务器2

Mon 192.168.1.4 3306 Mysql /var/lib/mysql 数据库管理服务器

Proxy 192.168.1.5 4040 Proxy  数据库代理(NLB)

node1 node2数据库服务器replication双 向 master-master 虚拟机有限，只能开4台， 因为node1与node2即做读又做写。

配置步骤：

Node1 node2 replication双向 master-master

Node1 node2 安装mmm并 配置mmm_regent.conf

Mon 安装mmm并配置mmm_mon.conf

proxy安装mysql-proxy

一、配置node1 node2数据库服务器replication双向 master-master

1、配置node1 同步

Mkdir /var/log/mysql

Chown mysql.mysql /var/log/mysql

my.cnf at db1 should have following options:

server-id = 1

log_bin = mysql-bin

master_host 192.168.1.2

master_port 3306

master_user replication

master_password slave

mysql&gt; grant replication slave on *.* to &amp;apos;replication&amp;apos;@&amp;apos;%&amp;apos; identified by &amp;apos;slave&amp;apos;;

show slave status\G;的结果：

Slave_IO_Running: Yes

Slave_SQL_Running: Yes

2、配置node2同步

Mkdir /var/log/mysql

Chown mysql.mysql /var/log/mysql

my.cnf at db1 should have following options:

server-id = 2

log_bin = mysql-bin

master_host 192.168.1.3

master_port 3306

master_user replication

master_password slave

mysql&gt; grant replication slave on *.* to &amp;apos;replication&amp;apos;@&amp;apos;%&amp;apos; identified by &amp;apos;slave&amp;apos;;

show slave status\G;的结果：

Slave_IO_Running: Yes

Slave_SQL_Running: Yes

二、安装部署MMM

目标主机：

Node1 192.168.1.2

Node2 192.168.1.3

Mon 192.168.1.4

1、安装mon主机软件包

前提：

- Algorithm-Diff-1.1902.tar.gz

- Proc-Daemon-0.03.tar.gz

RPM mysql-server-5.0.22-2.1

RPM mysql-5.0.22-2.1

RPM perl-DBD-MySQL-3.0007-1.fc6

- mmm-1.0.tar.bz2

先安裝2个perl的 包：

- Algorithm-Diff-1.1902.tar.gz

- Proc-Daemon-0.03.tar.gz

perl包的安裝過程 都是：

perl Makefile.PL

make

make test

make install

安裝mmm：

./install.pl

2、安装node1 node2 agent软件包（使用agent功 能）

mmm-1.0.tar.bz2

安裝mmm：

./install.pl

三台主机安装以上软件后进行配置过程

三：node1 node2 的agent配置过程

$cd /usr/local/mmm/etc

$cp examples/mmm_agent.conf.examples ../mmm_agent.conf

node1配置文件所需要修改的地方如下

cluster_interface eth0  －－配置IP所用的网卡

# Define current server id this db1

this db1

mode master

# For masters peer db2

Peer db2

# Cluster hosts addresses and access params

host db1

ip 192.168.1.2

port 3306

user rep_agent

password RepAgent

host db2

ip 192.168.1.3

port 3306

user rep_agent

password RepAgent

Node2配置文件所需要修改的地方如下

# Cluster interface

cluster_interface eth0  －－配置IP所用的网卡

# Define current server id this db1

this db2

mode master

# For masters peer db2

Peer db1

# Cluster hosts addresses and access params

host db1

ip 192.168.1.2

port 3306

user rep_agent

password RepAgent

host db2

ip 192.168.1.3

port 3306

user rep_agent

password RepAgent

设 置权限（node1/node2）

GRANT ALL PRIVILEGES on *.* to &amp;apos;rep_monitor&amp;apos;@&amp;apos;%&amp;apos; identified by &amp;apos;RepMonitor&amp;apos;;

四、修改mon主机的配置文件 所需要修改的地方如下：

文件位置：/usr/local/mmm/etc/mmm_mon.conf

# Cluster interface

cluster_interface eth0 －－－－配置IP所 用的网卡

# Cluster hosts addresses and access params

host db1 数据库名

ip 192.168.1.2 node1 ip

port 3306 node1 port

user rep_monitor node1 用户（供mon监控使用，该用户需要在db上添加）

password RepMonitor 对应密码

mode master 模式

peer db2 slave指定

host db2 意义同上

ip 192.168.1.3  

port 3306  

user rep_monitor  

password RepMonitor  

mode master  

peer db1  

#后续如有添加需求， 请按照上面格式填写

active_master_role writer

# Mysql Reader role 读规则

role reader

mode balanced 模式为均摊

servers db1,db2 规则覆盖db1 db2（如有更多slave 继续填写，别忘了ip ）

ip 192.168.1.7， 192.168.1.8 对应ip 虚拟的IP

# Mysql Writer role

role writer 写规则

mode exclusive 模式为独占

servers db1,db2 规则负载db1 db2

ip 192.168.1.9 两台数据库公用一个ip为写，采用HA模 式，默认db1使用，db1下线db2接管此ip

五： 测试MMM

在db1/db2上启动agent功能

/usr/local/mmm/scripts/init.d/mmm_agent start

在mon上启动mon功能

/usr/local/mmm/scripts/init.d/mmm_mon start

并 对mon进行检测

# mmm_control set_online db1

# mmm_control set_online db2

# mmm_control show  查看分配情况

正 常情况下:

# mmm_control show

Servers status:

db1(192.168.1.2): master/ONLINE. Roles: reader(192.168.1.7;), writer(192.168.1.9;)

db2(192.168.1.3): master/ONLINE. Roles: reader(192.168.1.8;)

stop 192.168.1.3

# mmm_control show

Servers status:

db1(192.168.1.2): master/ONLINE. Roles: reader(192.168.1.7;), writer(192.168.1.9;)

db2(192.168.1.3): master/REPLICATION_FAIL. Roles: None

检 测出1.3出了故障.

等 一会..进行了切换!因为读写是轮循的.这时写切到了3

# mmm_control show

Servers status:

db1(192.168.1.2): master/ONLINE. Roles: reader(192.168.1.7;)

db2(192.168.1.3): master/ONLINE. Roles: reader(192.168.1.8;), writer(192.168.1.9;)

Telnet 任何一个虚拟IP 3306都是通的

五、mysql_proxy与mysql MMM集成的必 要性

1、实现mysql数据库层的负载均衡

2、数据库节点实现HA动态切换

3、读写分离，降低主数据库负载

六、 安装mysql proxy

1、下载proxy代码包

从svn上获取最新代码

svn co [http://svn.mysql.com/svnpublic/mysql-proxy/trunk](http://svn.mysql.com/svnpublic/mysql-proxy/trunk) mysql-proxy

2、安装

编 译好的版本安装方法如下：

# tar zxf mysql-proxy-0.6.0-linux-rhas4-x86.tar.gz

# cd mysql-proxy-0.6.0-linux-rhas4-x86

#可以看到有2个目录

# ls

sbin share

# mv sbin/mysql-proxy /usr/local/sbin/

# ls share

mysql-proxy tutorial-constants.lua tutorial-packets.lua tutorial-rewrite.lua tutorial-warnings.lua tutorial-basic.lua tutorial-inject.lua tutorial-query-time.lua tutorial-states.lua

#将lua脚本放到/usr/local/share下，以 备他用

# mv share/mysql-proxy /usr/local/share/

#删除符号连接等垃圾代码

# strip /usr/local/sbin/mysql-proxy

proxy与MMM集成

连接接管 Proxy 192.168.1.5 4040

读操作 Db1 192.168.1.7:3306

Db2 192.168.1.8:3306

写操作 Db1/db2 192.168.1.9

默认db1先用，db1当机,db2接管

mysql-proxy -proxy-read-only-backend-addresses=192.168.1.7:3306 -proxy-read-only-backend-addresses=192.168.1.8:3306 -proxy-backend-addresses=192.168.1.9:3306  -proxy-lua-script=/usr/local/share/mysql-proxy/rw-splitting.lua &amp;

现在解释一下：

-proxy-backend-addresses=192.168.1.9:3306 指定mysql写主机的端口

-proxy-read-only-backend-addresses=192.168.1.7:3306 指定只读的mysql主机端口

-proxy-read-only-backend-addresses=192.168.1.8:3306 指定另一个只读 的mysql主机端口

-proxy-lua-script=/usr/local/share/mysql-proxy/rw-splitting.lua 指定lua脚本，在这里，使用的是rw-splitting脚 本，用于读写分离

完整的参数可以运行以下命令查看：

mysql-proxy -help-all

运行以下命令启动/停止/重启mysql proxy：

# /etc/init.d/mysql-proxy start

# /etc/init.d/mysql-proxy stop

# /etc/init.d/mysql-proxy restart

Ps -ef | grep mysql-proxy

七、测试结果

将web server 如apache 中部署的网站，数据库连接地址改为--〉proxy的ip端口为4040

1、往数据库db1里写入数据，查看2个数据库同步情况

2、使用mon服务器mmm_control show 查看状 态

简单的测试可以连接proxy 4040 查看读写情况
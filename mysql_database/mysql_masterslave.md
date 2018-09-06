## mysql master/slave {#mysql-master-slave}

[http://mysql-mmm.org/](http://mysql-mmm.org/)

主从工具

[http://tech.it168.com/a2009/0526/577/000000577322.shtml](http://tech.it168.com/a2009/0526/577/000000577322.shtml)

[http://blog.csdn.net/phphot/article/details/2255011](http://blog.csdn.net/phphot/article/details/2255011)

1、更新 magento local.xml

vim app/etc/local.xml

在default_setup下增加default_read

&lt;default_setup&gt;

   &lt;connection&gt;

       &lt;host&gt;&lt;![CDATA[master_host]]&gt;&lt;/host&gt;

       &lt;username&gt;&lt;![CDATA[magento_user]]&gt;&lt;/username&gt;

       &lt;password&gt;&lt;![CDATA[magento_pass]]&gt;&lt;/password&gt;

       &lt;dbname&gt;&lt;![CDATA[magento]]&gt;&lt;/dbname&gt;

       &lt;active&gt;1&lt;/active&gt;

   &lt;/connection&gt;

&lt;/default_setup&gt;

&lt;default_read&gt;

   &lt;connection&gt;

       &lt;use/&gt;

       &lt;host&gt;&lt;![CDATA[slave_host]]&gt;&lt;/host&gt;

       &lt;username&gt;&lt;![CDATA[magento_user]]&gt;&lt;/username&gt;

       &lt;password&gt;&lt;![CDATA[magento_pass]]&gt;&lt;/password&gt;

       &lt;dbname&gt;&lt;![CDATA[magento]]&gt;&lt;/dbname&gt;

       &lt;type&gt;pdo_mysql&lt;/type&gt;

       &lt;model&gt;mysql4&lt;/model&gt;

       &lt;initStatements&gt;SET NAMES utf8&lt;/initStatements&gt;

       &lt;active&gt;1&lt;/active&gt;

   &lt;/connection&gt;

&lt;/default_read&gt;

2 、主数据库配置

vim /etc/my.cnf

在mysqld下增加以下内容

[mysqld]

server-id       = 1

#log_bin         = mysql-bin

expire_logs_days    = 10

max_binlog_size     = 100M

binlog_do_db        = magento_demo//需要同步的数据库，如果没有本行，即表示同步所有的数据库

binlog-ignore-db=mysql //被忽略的，不需要同步的数据库

3、从数据库配置

vim /etc/my.cnf

在mysqld下增加以下内容

[mysqld]

server-id=2

log-bin=mysql-bin

master-host=192.168.1.2

master-user=username

master-password=password

master-port=3306

replicate-do-db=magento_demo

replicate-ignore-db=mysql

master-connect-retry=60

改完重启mysql服务器

4.在主服务器上为从服务器建立一个用户：

grant replication slave on *.* to &amp;apos;userdb&amp;apos;@&amp;apos;192.168.61.11&amp;apos; identified by &amp;apos;mysql&amp;apos;;

mysql&gt; GRANT ALL PRIVILEGES ON db.* TO adidas@&amp;apos;192.168.61.100&amp;apos; IDENTIFIED BY &amp;apos;password&amp;apos; WITH GRANT OPTION;

mysql&gt; FLUSH PRIVILEGES;

mysql&gt; show master status;

change master to master_host=&amp;apos;192.168.61.10&amp;apos;, master_user=&amp;apos;userdb&amp;apos;, master_password=&amp;apos;mysql&amp;apos;, master_log_file=&amp;apos;mysql-bin.000002&amp;apos;, master_log_pos=106;

mysql&gt; show slave status\G;
## autorun {#autorun}

mysql -uUSER -pPWD -e &quot;use mysql;select * from user&quot;

/usr/local/mysql/bin/mysql --socket=/SOHU/DB/server$port/mysql.sock dbname -e &quot;select * from table&quot;

mysql -h $mysqlHost -u$mysqlUser -p$mysqlPassword --databases $mysqlDB &lt; batchfile

可以用expect，密码的问题可以用一个密码文件，对这个文件加密，但是只是在读这个文件的时候看不到密码，执行的时候还是要解密的，还是能在终端看到
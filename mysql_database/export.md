## export {#export}

MySQL 查询结果导出到文件

选择某些行作为需要的数据

SELECT id,dbname FROM `index` into outfile &quot;d://aaa.txt&quot;;

一般大家都会用 &quot;SELECT INTO OUTFIL&quot;将查询结果导出到文件，但是这种方法不能覆盖或者添加到已经创建的文件，下文为您介绍的这种方法则很好地解决了此问题。

一般大家都会用 &quot;SELECT INTO OUTFIL&quot;将查询结果导出到文件，但是这种MySQL查询结果导出到文件方法不能覆盖或者添加到已经创建的文件。例如：

   mysql&gt; select 1 into outfile &amp;apos;/tmp/t1.txt&amp;apos;;  

   Query OK, 1 row affected (0.00 sec)  

   mysql&gt; select 1 into outfile &amp;apos;/tmp/t1.txt&amp;apos;;  

   ERROR 1086 (HY000): File &amp;apos;/tmp/t1.txt&amp;apos; already exists  

还可以使用另外一种方法：

   mysql&gt; pager cat &gt; /tmp/t1.txt  

   PAGER set to &amp;apos;cat &gt; /tmp/t1.txt&amp;apos;  

   mysql&gt; select 1;\! cat /tmp/t1.txt  

   1 row in set (0.00 sec)  

   +---+  

   | 1 |  

   +---+  

   | 1 |  

   +---+  

这样你能很方便的查询到2条sql的差异：

   mysql&gt; pager cat &gt; /tmp/t01.txt  

   PAGER set to &amp;apos;cat &gt; /tmp/t01.txt&amp;apos;  

   mysql&gt; select 12345 union select 67890;  

   2 rows in set (0.02 sec)  

   mysql&gt; pager cat &gt; /tmp/t02.txt  

   PAGER set to &amp;apos;cat &gt; /tmp/t02.txt&amp;apos;  

   mysql&gt; select 12345 union select 67891;  

   2 rows in set (0.00 sec)  

   mysql&gt; \! vimdiff -o  /tmp/t0[12].txt  

   2 files to edit  

     +-------+  

     | 12345 |  

     +-------+  

     | 12345 |  

     | 67890 |                                                                                                                  

     +-------+                                                                                                                

   /tmp/t01.txt  

     +-------+  

     | 12345 |  

     +-------+  

     | 12345 |  

     | 67891 |                                                                                                                  

     +------+                                                                                                          

   /tmp/t02.txt
# mysql线上常见故障排查集锦

 **Author:** [兰堂红烛]

 **Link:** [https://zhuanlan.zhihu.com/p/691957928]

## 一、mysqcpu高  
cpu高，**基本是读写磁盘和排序问题**，读写磁盘是因为没有使用索引，各种语句导致的排序问题会导致buffer不够写磁盘。

1. wait\_timeout造成：
1. [https://blog.csdn.net/Beucejiang/article/details/122269910](https://blog.csdn.net/Beucejiang/article/details/122269910)：解决办法：wait\_timeout默认是8h，改成120s。

3. sql占用资源多：show full processlist; #显示哪些线程正在运行。或者select \* from information\_schema.`PROCESSLIST` where info is not null;
4. 高峰期连接数激增
1. 限制连接数：也有可能是每个 sql 消耗资源并不多，但是突然之间，有大量的 session 连进来导致 cpu 飙升，这种情况就需要跟应用一起来分析为何连接数会激增，再做出相应的调整，比如说限制连接数等。

6. 大量io造成：
1. **全表扫描造成大量io**：
1. [https://blog.csdn.net/yabingshi\_tech/article/details/101675792](https://blog.csdn.net/yabingshi\_tech/article/details/101675792)。查看慢查询（sql超过500ms/1s/2s的），查**看执行计划，发现走了全表扫描，该表数据量有200万。**执行以下语句能看到有**几十条执行时间在1秒**的查询
2. [https://zhuanlan.zhihu.com/p/651665735](https://zhuanlan.zhihu.com/p/651665735) ：select count(\*) 带了多个子查询，某个子查询的where条件没有索引，导致扫描数据很多达到了17w条。

3. **排序导致磁盘写文件：**
1. **这类问题的解决思路：**
1. **优化sql，减小数据量，避免写磁盘**
2. **增大sort\_buffer\_pool\_size(默认2M)**
3. **使用联合索引，走索引覆盖using index**

3. **Using filesort** 文件排序：可能导致写文件。
1. 语句中有**order by** most\_top ，explan Using filesort（Using index说明使用了覆盖索引排序）导致写磁盘。另外排序while循环比较运算消耗cpu高。
2. --filesort是MySQL中的一个排序操作，它不一定使用磁盘文件进行排序，而是根据**sort\_buffer\_size**参数的大小决定是否在内存中完成排序1234。filesort使用的算法是QuickSort和mergesort，它会对需要排序的记录生成元数据进行分块排序，然后再合并块2。filesort会影响SQL的性能，因为它表示没有使用索引的排序。**当排序字段的取值分布较为均匀时，Using filesort会更加高效。 而当排序字段的取值分布不均匀时，比如存在大量的重复值或者数据集较大时，Using filesort可能需要更多的内存或者磁盘空间，从而导致性能下降**。
3. --解决办法：**加联合索引**，或者把使用**where条件把数据集弄**小一点。

5. **Using temporary**：可能导致写文件。
1. **[https://zhuanlan.zhihu.com/p/691957928/</b](https://zhuanlan.zhihu.com/p/691957928/</b) ：多个表连接查询时，使用distinct去重，导致了重新排序。解决：去掉distinct，业务层解决。**
2. **多表连接时候order by** 导致useing temploray。order by产生了using temploray，可能是内存里的临时表，数据多了以后可能是磁盘上的临时表。

7. group by导致隐含排序：5.7以前，group by 会进行隐式排序（见[https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)），5.8以后去掉了。5.7以前，使用explain可以看到group by不加order bynull 会走using filesort，加上则没有filesort了。

8. 锁造成（*行锁冲突、行锁等待*）
1. 锁冲突

10. 后台任务

## 二、Too many connections  
## 三、死锁  
update会加哪些锁：[https://www.maoyingdong.com/mysql-update-sql-locking/](https://www.maoyingdong.com/mysql-update-sql-locking/)

如果表中有数据id为1、3、5、7?、8、9

1. update ... where id=7 : 如果7存在，加行所，如果不存在，加间隙锁(5,8）。
2. update ... where id>=7：行锁7、8、间隙锁`(8, +∞)`

死锁产生的原因：[https://www.cnblogs.com/hanease/p/15955245.html](https://www.cnblogs.com/hanease/p/15955245.html)

  


  


## 解决手段：  
1. 实例诊断报告：CALL sys.diagnostics(120, 30, 'current');
2. MySQL提供了日志分析工具`mysqldumpslow` 分析慢查询
3. show profiler 分析sql耗时: [https://blog.csdn.net/huangliang0703/article/details/49891621](https://blog.csdn.net/huangliang0703/article/details/49891621)

  


## 优化建议  
1. 不要在MYSQL上经常使用ORDER BY
2. 使用GROUP BY 的时候后面还建议带一个 order by null.

## Percona-toolkit 使用教程

### 一、 percona-toolkit 简介
percona-toolkit 是一组高级命令行工具的集合,用来执行各种通过手工执行非常复
杂和麻烦的 mysql 任务和系统任务,这些任务包括:
 检查 master 和 slave 数据的一致性
 有效地对记录进行归档
 查找重复的索引
 对服务器信息进行汇总
 分析来自日志和 tcpdump 的查询
 当系统出问题的时候收集重要的系统信息
percona-toolkit 源自 Maatkit 和 Aspersa 工具,这两个工具是管理 mysql 的最有名的
工具,现在 Maatkit 工具已经不维护了,请大家还是使用 percona-toolkit 吧!这些
工具主要包括开发、性能、配置、监控、复制、系统、实用六大类,作为一个优秀
的 DBA,里面有的工具非常有用,如果能掌握并加以灵活应用,将能极大的提高工
作效率。

### 二. 原理
1、在InnoDB内部会维护一个redo/undo日志文件，也可以叫做事务日志文件。事务日志会存储每一个InnoDB表数据的记录修改。当InnoDB启动时，InnoDB会检查数据文件和事务日志，并执行两个步骤：它应用（前滚）已经提交的事务日志到数据文件，并将修改过但没有提交的数据进行回滚操作。

2、Xtrabackup在启动时会记住log sequence number（LSN），并且复制所有的数据文件。复制过程需要一些时间，所以这期间如果数据文件有改动，那么将会使数据库处于一个不同的时间点。这时，xtrabackup会运行一个后台进程，用于监视事务日志，并从事务日志复制最新的修改。Xtrabackup必须持续的做这个操作，是因为事务日志是会轮转重复的写入，并且事务日志可以被重用。所以xtrabackup自启动开始，就不停的将事务日志中每个数据文件的修改都记录下来。

3、上面就是xtrabackup的备份过程。接下来是准备（prepare）过程，在这个过程中，xtrabackup使用之前复制的事务日志，对各个数据文件执行灾难恢复（就像mysql刚启动时要做的一样）。当这个过程结束后，数据库就可以做恢复还原了，这个过程在xtrabackup的编译二进制程序中实现。程序innobackupex可以允许我们备份MyISAM表和frm文件从而增加了便捷和功能。Innobackupex会启动xtrabackup，直到xtrabackup复制数据文件后，然后执行FLUSH TABLES WITH READ LOCK来阻止新的写入进来并把MyISAM表数据刷到硬盘上，之后复制MyISAM数据文件，最后释放锁。

4、备份MyISAM和InnoDB表最终会处于一致，在准备（prepare）过程结束后，InnoDB表数据已经前滚到整个备份结束的点，而不是回滚到xtrabackup刚开始时的点。这个时间点与执行FLUSH TABLES WITH READ LOCK的时间点相同，所以myisam表数据与InnoDB表数据是同步的。类似oracle的，InnoDB的prepare过程可以称为recover（恢复），myisam的数据复制过程可以称为restore（还原）。

5、Xtrabackup 和 innobackupex这两个工具都提供了许多前文没有提到的功能特点。手册上有对各个功能都有详细的介绍。简单介绍下，这些工具提供了如流（streaming）备份，增量（incremental）备份等，通过复制数据文件，复制日志文件和提交日志到数据文件（前滚）实现了各种复合备份方式。

### 三，安装

```
CentOS 7
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.10/binary/redhat/7/x86_64/Percona-XtraBackup-2.4.10-r3198bce-el7-x86_64-bundle.tar

Ubuntu16.04
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.10/binary/debian/xenial/x86_64/percona-xtrabackup-24_2.4.10-1.xenial_amd64.deb
```

### 四,备份与恢复

```
修改my.cnf以作增量备份
```
[mysqld] 
datadir=/var/lib/mysql
 
innodb_data_home_dir = /data/mysql/ibdata
innodb_log_group_home_dir = /data/mysql/iblogs
innodb_data_file_path=ibdata1:10M;ibdata2:10M:autoextend
innodb_log_files_in_group = 2
innodb_log_file_size = 1G
```

备份
```
//全部数据库备份
innobackupex --user=root --password=123456 /data/backup/
 
//单数据库备份
innobackupex --user=root --password=123456 --database=backup_test /data/backup/
 
//多库
innobackupex--user=root --password=123456 --include='dba.*|dbb.*' /data/backup/
 
//多表
innobackupex --user=root --password=123456 --include='dba.tablea|dbb.tableb' /data/backup/
 
//数据库备份并压缩
log=zztx01_`date +%F_%H-%M-%S`.log
db=zztx01_`date +%F_%H-%M-%S`.tar.gz
innobackupex --user=root --stream=tar /data/backup  2>/data/backup/$log | gzip 1> /data/backup/$db
//不过注意解压需要手动进行，并加入 -i 的参数，否则无法解压出所有文件,疑惑了好长时间
 
//如果有错误可以加上  --defaults-file=/etc/my.cnf
```

还原
```
service mysqld stop
mv /data/mysql /data/mysql_bak && mkdir -p /data/mysql
 
//--apply-log选项的命令是准备在一个备份上启动mysql服务
innobackupex --defaults-file=/etc/my.cnf --user=root --apply-log /data/backup/2015-09-18_16-35-12
 
//--copy-back 选项的命令从备份目录拷贝数据,索引,日志到my.cnf文件里规定的初始位置
innobackupex --defaults-file=/etc/my.cnf --user=root --copy-back /data/backup/2015-09-18_16-35-12
 
chown -R mysql.mysql /data/mysq
service mysqld start
```
## 五,正能量备份与还原

1.创建测试数据库和表
```
create database backup_test; //创建库
 
CREATE TABLE `backup` ( //创建表
`id` int(11) NOT NULL AUTO_INCREMENT ,
`name` varchar(20) NOT NULL DEFAULT '' ,
`create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ,
`del` tinyint(1) NOT NULL DEFAULT '0',
PRIMARY KEY (`id`)
) ENGINE=myisam DEFAULT CHARSET=utf8 AUTO_INCREMENT=1 ;
```

2.增量备份
```
#--incremental：增量备份的文件夹
#--incremental-dir：针对哪个做增量备份
 
//第一次备份
mysql> INSERT INTO backup (name) VALUES ('xx'),('xxxx'); //插入数据
innobackupex  --user=root --incremental-basedir=/data/backup/2015-09-18_16-35-12 --incremental /data/backup/
 
//再次备份
mysql> INSERT INTO backup (name) VALUES ('test'),('testd'); //在插入数据
innobackupex --user=root --incremental-basedir=/data/backup/2015-09-18_18-05-20 --incremental /data/backup/
```

3. 查看增量备份记录文件
```
[root@localhost 2015-09-18_16-35-12]# cat xtrabackup_checkpoints //全备目录下的文件
backup_type = full-prepared
from_lsn = 0 //全备起始为0
to_lsn = 23853959
last_lsn = 23853959
compact = 0
 
[root@localhost 2015-09-18_18-05-20]# cat xtrabackup_checkpoints //第一次增量备份目录下的文件
backup_type = incremental
from_lsn = 23853959
to_lsn = 23854112
last_lsn = 23854112
compact = 0
 
[root@localhost 2015-09-18_18-11-43]# cat xtrabackup_checkpoints //第二次增量备份目录下的文件
backup_type = incremental
from_lsn = 23854112
to_lsn = 23854712
last_lsn = 23854712
compact = 0
```
增量备份做完了后, 把backup_test这个数据库删除掉, drop database backup_test;这样可以对比还原后

4.增量还原
分为两个步骤
a.prepare
```
1
innobackupex --apply-log /path/to/BACKUP-DIR
```
此时数据可以被程序访问使用；可使用—use-memory选项指定所用内存以加快进度，默认100M；

b.recover
```
1
innobackupex --copy-back /path/to/BACKUP-DIR
```
从my.cnf读取datadir/innodb_data_home_dir/innodb_data_file_path等变量

先复制MyISAM表，然后是innodb表，最后为logfile；--data-dir目录必须为空

开始合并
```
innobackupex --apply-log --redo-only /data/backup/2015-09-18_16-35-12
innobackupex --apply-log --redo-only --incremental /data/backup/2015-09-18_16-35-12 --incremental-dir=/data/backup/2015-09-18_18-05-20
innobackupex --apply-log --redo-only --incremental /data/backup/2015-09-18_16-35-12 --incremental-dir=/data/backup/2015-09-18_18-11-43
 
#/data/backup/2015-09-18_16-35-12 全备份目录
#/data/backup/2015-09-18_18-05-20 第一次增量备份产生的目录
#/data/backup/2015-09-18_18-11-43 第二次增量备份产生的目录
```

恢复数据
```
service mysqld stop
innobackupex --copy-back /data/backup/2015-09-18_16-35-12
service mysqld start
```

## 六, 常用参数说明
```
--defaults-file
同xtrabackup的--defaults-file参数

--apply-log
对xtrabackup的--prepare参数的封装

--copy-back
做数据恢复时将备份数据文件拷贝到MySQL服务器的datadir ；

--remote-host=HOSTNAME
通过ssh将备份数据存储到进程服务器上；

--stream=[tar]
备 份文件输出格式, tar时使用tar4ibd , 该文件可在XtarBackup binary文件中获得.如果备份时有指定--stream=tar, 则tar4ibd文件所处目录一定要在$PATH中(因为使用的是tar4ibd去压缩, 在XtraBackup的binary包中可获得该文件)。
在 使用参数stream=tar备份的时候，你的xtrabackup_logfile可能会临时放在/tmp目录下，如果你备份的时候并发写入较大的话 xtrabackup_logfile可能会很大(5G+)，很可能会撑满你的/tmp目录，可以通过参数--tmpdir指定目录来解决这个问题。

--tmpdir=DIRECTORY
当有指定--remote-host or --stream时, 事务日志临时存储的目录, 默认采用MySQL配置文件中所指定的临时目录tmpdir

--redo-only --apply-log组,
强制备份日志时只redo ,跳过rollback。这在做增量备份时非常必要。

--use-memory=#
该参数在prepare的时候使用，控制prepare时innodb实例使用的内存量

--throttle=IOS
同xtrabackup的--throttle参数

--sleep=是给ibbackup使用的，指定每备份1M数据，过程停止拷贝多少毫秒，也是为了在备份时尽量减小对正常业务的影响，具体可以查看ibbackup的手册 ；

--compress[=LEVEL]
对备份数据迚行压缩，仅支持ibbackup，xtrabackup还没有实现；

--include=REGEXP
对 xtrabackup参数--tables的封装，也支持ibbackup。备份包含的库表，例如：--include="test.*"，意思是要备份 test库中所有的表。如果需要全备份，则省略这个参数；如果需要备份test库下的2个表：test1和test2,则写 成：--include="test.test1|test.test2"。也可以使用通配符，如：--include="test.test*"。

--databases=LIST
列出需要备份的databases，如果没有指定该参数，所有包含MyISAM和InnoDB表的database都会被备份；

--uncompress
解压备份的数据文件，支持ibbackup，xtrabackup还没有实现该功能；

--slave-info,
备 份从库, 加上--slave-info备份目录下会多生成一个xtrabackup_slave_info 文件, 这里会保存主日志文件以及偏移, 文件内容类似于:CHANGE MASTER TO MASTER_LOG_FILE='', MASTER_LOG_POS=0

--socket=SOCKET
指定mysql.sock所在位置，以便备份进程登录mysql.
```

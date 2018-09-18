

1.安装 yum 源
sudo yum install http://yum.postgresql.org/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm

（从 http://yum.postgresql.org/repopackages.php 获取该地址）

2.安装 postgresql95-server 和 postgresql95-contrib
sudo yum install postgresql95-server postgresql95-contrib

安装后，可执行文件在 /usr/pgsql-9.5/bin/， 数据和配置文件在 /var/lib/pgsql/9.5/data/

3.初始化数据:
sudo /usr/pgsql-9.5/bin/postgresql95-setup initdb

4.默认不支持密码认证，修改 pg_hab.conf 将 ident 替换为 md5 （可选）
sudo vim /var/lib/pgsql/9.5/data/pg_hba.conf

5.启动服务：
sudo systemctl start postgresql-9.5.service

6.修改用户密码
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'postgres';"

或：

psql -c "ALTER USER postgres PASSWORD 'postgres';"





7.配置远程访问
vim /var/lib/pgsql/9.5/data/postgresql.conf

 修改#listen_addresses = 'localhost'  为  listen_addresses='*'

*表示所有地址，也可以是指定的单个IP

vim /var/lib/pgsql/9.5/data/pg_hba.conf

修改如下内容，加入一行：

# IPv4 local connections:
host    all             all         0.0.0.0/0               md5


修改配置后需要重启：

systemctl restart postgresql-9.5.service

如果还是连接不通，可能需要打开防火墙，配置完成后可以用Navicat for PostgreSQL连接数据库，其操作跟MYSQL一样。

8.设置为开机启动:
sudo systemctl enable postgresql-9.5.service

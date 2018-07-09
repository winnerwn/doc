##创建表
```
create table `log_eris`(
    `id` int(10) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `groups` varchar(50) NOT NULL,
    `keys` varchar(100) NOT NULL,
    `values` varchar(100) NOT NULL,
    `description` varchar(100)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

创建触发器(监控表增删改然后同步到一张日志表里面)

```
mysql> DELIMITER ||
CREATE TRIGGER monitor_log1 AFTER INSERT ON s_sys_config FOR EACH ROW
BEGIN
INSERT INTO log_eris(`groups`, `keys`, `values`, `description`)
VALUES(`new.groups`,`new.keys`,`new.values`,`new.description`);
END
||
mysql> DELIMITER ;
```

```
mysql> DELIMITER ||
CREATE TRIGGER monitor_log2 AFTER UPDATE ON s_sys_config FOR EACH ROW
BEGIN
INSERT INTO log_eris(`groups`, `keys`, `values`, `description`)
VALUES(`new.groups`,`new.keys`,`new.values`,`new.description`);
END
||
mysql> DELIMITER ;
```

```
mysql> DELIMITER ||
CREATE TRIGGER monitor_log3 AFTER DELETE ON s_sys_config FOR EACH ROW
BEGIN
INSERT INTO log_eris(`groups`, `keys`, `values`, `description`)
VALUES(`old.groups`,`old.keys`,`old.values`,`old.description`);
END
||
mysql> DELIMITER ;
```

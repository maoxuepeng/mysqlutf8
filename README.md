# mysql utf8 how to

## 其他人都是不正确的
使用mysql容器镜像可以很快速的运行mysql，免去了传统的虚拟机安装方式的繁琐配置。
但是使用[官方的mysql镜像](https://hub.docker.com/_/mysql/)，你会遇到中文乱码的问题，原因是官方镜像的字符集默认值不是utf8。
这时候你去google，会找到一些文章，如[这个哥们的](https://github.com/dnhsoft/docker-mysql-utf8)，但是你按照他的做法，还是会乱码，这时候你进入mysql查看字符集，发现是这样的：
```
show variables like "%character%";

+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_database   | latin1                           |
| character_set_filesystem | binary                           |
| character_set_results    | utf8                             |
| character_set_server     | latin1                           |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysqlarsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.00 sec)
```

## 这里才是正确的
好吧，我费了不少力气，才找到正确方法。

首先查看官方镜像说明，有关于utf8设置的参数：
```
Many configuration options can be passed as flags to mysqld. This will give you the flexibility to customize the container without needing a cnf file. For example, if you want to change the default encoding and collation for all tables to use UTF-8 (utf8mb4) just run the following:
    $ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```
如果只是设置这两个参数是不够的，你会得到这样的配置：
```
mysql> show variables like "%character%";
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | latin1                           |
| character_set_connection | latin1                           |
| character_set_database   | utf8                             |
| character_set_filesystem | binary                           |
| character_set_results    | latin1                           |
| character_set_server     | utf8                             |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysql/share/charsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.00 sec)
```
你会发现上面那个哥们的与官方的这个指导取并集，就是正确的结果了，所以方法就很简单了。查看dockerfile。
正确的配置：
```
mysql> show variables like "%character%";
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_database   | utf8                             |
| character_set_filesystem | binary                           |
| character_set_results    | utf8                             |
| character_set_server     | utf8                             |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysql/share/charsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.01 sec)
```

# Wanna-Build 搭建


## 基础环境

Debian 12 x86

### 软件包依赖安装

```
apt install libdbi-perl libyaml-libyaml-perl libhash-merge-perl libstring-format-perl libtimedate-perl libyaml-tiny-perl libdpkg-perl libdbd-pg-perl libany-uri-escape-perl dctrl-tools moreutils dose-builddebcheck

apt install dose-distcheck

```

### 文件夹设置

```
mkdir -p /srv

git clone https://salsa.debian.org/wb-team/wanna-build.git /srv/wanna-build

ln -s /srv/wanna-build/bin/wanna-build /usr/local/bin/wanna-build
```


原文批注：Note that /srv is hardcoded in wanna-build.


## 安装数据库 


安装 postgresql

```

apt-get install postgresql-15 postgresql-15-debversion

```

创建数据库以及用户

```
su postgres

psql

CREATE DATABASE wannadb;

CREATE USER wbadm WITH PASSWORD 'wannapass';

GRANT ALL PRIVILEGES ON DATABASE wannadb to wbadm;

ALTER ROLE wbadm CREATEROLE;

```

执行完毕后退出上述环境，进行数据表导入。

```
psql -d wannadb -f /srv/wanna-build/schema/roles.sql
psql -d wannadb -f /srv/wanna-build/schema/main-tables.sql
```

生成对应架构的表：
```
cd /srv/wanna-build/schema/

./arches-tables.sh $ARCH | psql -d wannadb
```

创建wbadm系统用户

```
su

adduser --disabled-password --gecos ""  wbadm

mkdir -p /srv/wanna-build/tmp

chmod 750 /srv/wanna-build/tmp/

chown wbadm /srv/wanna-build/tmp/

```


配置 PostgreSQL 服务
```
cp /usr/share/postgresql/15/pg_service.conf.sample /etc/postgresql-common/pg_service.conf
```

查看 /etc/postgresql-common/pg_service.conf 确保包含下面的内容

```
[wanna-build]
dbname=wannadb
user=wbadm

[wanna-build-privileged]
dbname=wannadb
user=wbadm
```

放行数据库访问权限

```
vim /etc/postgresql/*/main/pg_hba.conf
```

加入下面的内容：

```
local   all             wbadm                                   trust
```


## 安装数据库（docker） 该方法存在严重问题，暂时请勿使用 



docker pull postgres

docker run --name wbpostgres -e POSTGRES_USER=wb -e POSTGRES_PASSWORD=niconicowb@buildtest -e POSTGRES_DB=mydatabase -p 5432:5432 -d postgres

psql -h 127.0.0.1 -p 5432 -U wb -d postgres

输入密码

Then create a database and user for wanna-build.


CREATE DATABASE wannadb;
CREATE USER wbadm WITH PASSWORD 'wannapass';
GRANT ALL PRIVILEGES ON DATABASE wannadb to wbadm;
ALTER ROLE wbadm CREATEROLE;


写入数据库信息


Import roles.sql and main-tables.sql.


psql -h 127.0.0.1 -p 5432 -U wb -d postgres -f /srv/wanna-build/schema/roles.sql
psql -h 127.0.0.1 -p 5432 -U wb -d postgres -f /srv/wanna-build/schema/main-tables.sql

*需要使用密码


## 运行


如果你的安装正常，现在直接敲 wanna-build 可以看到相关输出


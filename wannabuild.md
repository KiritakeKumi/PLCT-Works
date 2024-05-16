# Wanna-Build 搭建


## 基础环境

### 软件包依赖安装


apt install libdbi-perl libyaml-libyaml-perl libhash-merge-perl libstring-format-perl libtimedate-perl libyaml-tiny-perl libdpkg-perl libdbd-pg-perl libany-uri-escape-perl dctrl-tools moreutils dose-builddebcheck

apt install dose-distcheck

### 文件夹设置

mkdir -p /srv
git clone https://salsa.debian.org/wb-team/wanna-build.git /srv/wanna-build
ln -s /srv/wanna-build/bin/wanna-build /usr/local/bin/wanna-build

原文批注：Note that /srv is hardcoded in wanna-build.


## 安装数据库

官方推荐 PostgreSQL ，此处也使用 PostgreSQL .

此处和官方方法略微差异 PostgreSQL 采用docker安装.

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
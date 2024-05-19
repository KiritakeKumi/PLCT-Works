# Wanna-Build 搭建 (docker)


基础环境：

## 安装主程序

## 安装数据库（docker） 该方法存在严重问题，暂时请勿使用 



docker pull postgres

docker run --name wbpostgres -e POSTGRES_USER=wb -e POSTGRES_PASSWORD=niconicowb@buildtest -e POSTGRES_DB=mydatabase -p 5432:5432 -d postgres


docker run --name wannabuild –net=host -d debian:bookworm

psql -h 127.0.0.1 -p 5432 -U wb -d postgres

输入密码

Then create a database and user for wanna-build.


CREATE DATABASE wannadb;
CREATE USER wbadm WITH PASSWORD 'wannapass';
GRANT ALL PRIVILEGES ON DATABASE wannadb to wbadm;
ALTER ROLE wbadm CREATEROLE;


写入数据库信息


Import roles.sql and main-tables.sql.


psql -h 127.0.0.1 -p 5432 -U wb -d wannadb -f /srv/wanna-build/schema/roles.sql
psql -h 127.0.0.1 -p 5432 -U wb -d wannadb -f /srv/wanna-build/schema/main-tables.sql

*需要使用密码

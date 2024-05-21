# Wanna-Build+Buildd+Sbuild 搭建


## 基础环境&简述

Debian 12 x86

wanna-build 可以和 Buildd  Sbuild 运行在不同的服务器上。

## wanna-build 安装

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

/etc/init.d/postgresql restart


## 运行


如果你的安装正常，现在直接敲 wanna-build 可以看到相关输出



## sbuild 及 schroot 安装

官方文档： https://wiki.debian.org/sbuild

从仓库安装
```
sudo apt install sbuild
```
添加用户组
```
sudo adduser $(whoami) sbuild
```
生成 chroot 环境
```
sudo sbuild-createchroot --include=eatmydata,ccache,gnupg stable /srv/chroot/stable http://deb.debian.org/debian/
```

配置 .sbuildrc
```
nano ~/.sbuildrc
```
```
$distribution = 'stable';
$purge_build_deps = 'successful';
$build_arch_all = 1;
$run_lintian = 1;
```
这些配置设置了默认的发行版，定义了何时清理构建依赖，是否构建独立架构的包，以及是否在构建后运行 lintian 检查。

测试功能
```
sbuild -d stable your_package.dsc
```

## buildd 安装


从仓库安装
```
apt install sbuild buildd sudo schroot debootstrap
```
配置用户组
```
usermod -a -G sbuild buildd
```

修改配置文件
```
vim /etc/buildd/buildd.conf
```

将下面的内容写入文件，wanna_build_ssh_host 请根据实际情况填写，如就在同一台服务器上无需改动。
```
$build_arch = 'amd64';
$distribution = 'sid';
@distributions = (
        {
                dist_name => ['sid'],
                built_architecture => 'amd64',
                wanna_build_ssh_host => "127.0.0.1",
                wanna_build_ssh_user => "buildd",
                wanna_build_db_user => 'wbadm',
                dupload_local_queue_dir => "upload",
                logs_mailed_to => $admin_mail,
        }
);
$host_arch = 'amd64';
$daemon_log_file = '/var/log/buildd.log';
$idle_sleep_time = 300;
$verbose = 16;
$admin_mail = 'root';
$log_queued_messages = 1;
$apt_get = 'apt-get';
$mailprog = '/usr/sbin/sendmail';
$ssh = 'ssh';
$sudo = 'sudo';
$upload_queues = [
        {
                dupload_local_queue_dir => "upload",
                dupload_archive_name => "anonymous-ftp-master",
        }
];
$statistics_mail = 'root';
$wanna_build_built_architecture = 'amd64';
$wanna_build_db_user = 'wbadm';
$wanna_build_ssh_host = 'localhost';
$wanna_build_ssh_socket = '/var/buildd.sock';
$wanna_build_ssh_user = 'buildd';
1;
```
生成log文件并配置权限
```
touch /var/log/buildd.log
chown buildd:buildd /var/log/buildd.log
```
生成ssh密钥
```
su - buildd
ssh-keygen
```
测试连接，如连接正常退出即可
```
cd /var/lib/buildd/.ssh/
cat id_rsa.pub > authorized_keys
ssh 127.0.0.1
```
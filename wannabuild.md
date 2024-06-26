# Wanna-Build+Buildd+Sbuild 搭建


## 基础环境&简述

Debian 12 x86

wanna-build 可以和 Buildd  Sbuild 运行在不同的服务器上。


在 Debian 的包编译和构建过程中，wanna-build、Buildd 和 Sbuild 是三个关键的组件，它们相互协作以确保软件包的正确构建和维护。这里简要介绍它们的功能和关系：

wanna-build：

wanna-build 是一个数据库系统，用于追踪软件包的构建状态。它记录了哪些包需要被构建、已经成功或失败构建的包，以及这些包是否需要重新构建。wanna-build 管理着与构建相关的各种数据，比如版本、构建依赖和构建日志的位置。

Buildd：

Buildd 是指构建守护进程，其作用是周期性地从 wanna-build 数据库获取需要构建的包列表，并启动构建过程。Buildd 会处理软件包的构建队列，并根据 wanna-build 数据库中的信息决定哪些包需要构建或重新构建。

Sbuild：

Sbuild 是实际进行包构建的工具。它为 Debian 软件包的构建提供了一个干净的 chroot 环境。Sbuild 负责在这个隔离的环境中配置必要的依赖，编译软件包，并生成可安装的包文件。它确保构建过程不会受到主机环境的干扰，从而提高包构建的可重复性和可靠性。

总结起来，wanna-build 管理和跟踪包的构建状态，Buildd 是从 wanna-build 获取数据并管理构建队列的守护进程，而 Sbuild 是在隔离的环境中实际执行构建任务的工具。这三者共同工作，确保 Debian 软件包的有效、高效构建。

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


### 安装数据库 


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


### 测试运行


如果你的安装正常，现在直接敲 wanna-build 可以看到相关输出


接下来还需要向数据库中加入一些数据

写入支持的架构和发行版。

```
INSERT INTO distributions(distribution,build_dep_resolver) VALUES ('sid','apt');

INSERT INTO architectures(architecture) VALUES ('amd64'),('i386');

INSERT INTO distribution_architectures(distribution,architecture,archive) VALUES ('sid','amd64','debian'),('sid','i386','debian');

INSERT INTO locks(distribution,architecture) VALUES ('sid','amd64'),('sid','i386');
```

设置发行版的别名。

```
INSERT INTO distribution_aliases(distribution,alias) VALUES('sid','unstable');
```


当设置本地wanna-build数据库时，我们必须为每个套件提供一个当前的“ Packages-arch-specific ”文件，这是wanna-build操作所需要的。它的路径在 /srv/wanna-build/triggers/common 中的变量 $PAS_BASE 和 $PAS_FILE 中定义。使用预定义路径时，可以使用以下命令安装该文件：

```
su

SUITE=sid

mkdir -p /srv/buildd.debian.org/web/quinn-diff/${SUITE}

cd /srv/buildd.debian.org/web/quinn-diff/${SUITE}/

wget https://buildd.debian.org/quinn-diff/${SUITE}/
Packages-arch-specific
```


如果从本地存储库获取包和源，可以使用 trigger.local 脚本，需要将脚本中的 REPOSITORY 变量更改为存储库所在的位置。

trigger.local ：https://wiki.debian.org/DebianWannaBuildInfrastructureOnOneServer?action=AttachFile&do=view&target=trigger.local

```
# copy trigger.local into /srv/wanna-build/triggers

su

chmod a+x /srv/wanna-build/triggers/trigger.local

su wbadm

cd /srv/wanna-build/triggers

./trigger.local

```



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

## dak 安装 （Debian Archive Kit）


官方文档：https://salsa.debian.org/ftp-team/dak/-/tree/master/setup


基础软件包安装

```
apt install python3-psycopg2 python3-sqlalchemy python3-apt gnupg dpkg-dev lintian binutils-multiarch python3-yaml less python3-ldap python3-pyrss2gen python3-rrdtool symlinks python3-debian python3-debianbts python3-tabulate
```

```
cd /srv

git clone https://salsa.debian.org/ftp-team/dak.git

cd dak
```
设置 dak 用户：
```
sudo addgroup ftpmaster

sudo adduser dak --disabled-login --ingroup ftpmaster --shell /bin/bash
```

设置 dak 目录：
```
sudo mkdir /etc/dak

```

创建链接（实际文件将由安装脚本创建）：

sudo ln -s /srv/dak/etc/dak.conf /etc/dak/dak.conf


此脚本完成其余工作。它使用 init_vars 中设置的通用变量，如果需要，可以自定义这些变量：
```
setup/dak-setup.sh
```

添加系统变量：
```
export PATH="/srv/dak/bin:${PATH}"
```


## 启动

```
su - wbadm

cd /srv/wanna-build/triggers/

./trigger.local

exit
```

trigger.local 文件请根据实际情况来，此处仅为本地测试用

```
/etc/init.d/buildd start
```
启动buildd进程

请通过以下文件查看日志

```
cat /var/log/buildd.log


```


## 后续

### wanna-build 多repo共存

官方对于wanna-build的文档仍然停留在13年前，内容主要是各个构建状态的情况，并没有提到程序部署的具体细节 

https://salsa.debian.org/wb-team/wanna-build/-/blob/master/states.txt

目前已知的多repo方法为为每个仓库创建一个单独的数据库，以隔离不同仓库的数据。

启用多个wanna-build 实例 配置不同的数据库连接信息来实现多repo

关于数据库相关操作的官方阐述：https://salsa.debian.org/wb-team/wanna-build/-/blob/master/schema/README

但是此处并没有说明如何操作，数据库相关操作也并没有像Buildd通过独立文件那样方便控制，或许仍需容器化来实现多个实例


### wanna-build 运行检测rebuild


wanna-build本质是几个perl和python脚本组成的脚本群，核心的数据均存储在数据库中，也就是说构建成功与否均来自后端 Buildd  Sbuild 的指令。

（官方阐述在此处：https://salsa.debian.org/wb-team/wanna-build/-/blob/master/states.txt）

同时上文给出了各个构建状态，大体分为初始构建和最终状态，构建失败（Failed）状态由Buildd来给出

首先是列出所有标记为构建失败的包
```
wanna-build --list=failed
```
更新这些包的状态，使其从“Failed”变为“Needs-Build”
```
wanna-build --requeue --dist=unstable package-name
```
package-name 需要替换为实际的包名，--dist=unstable 参数指定发行版，状态更新后，包将自动进入 Buildd 管理的构建队列等待重新构建。Buildd 作为构建守护进程，会周期性地检查哪些包处于“Needs-Build”状态。


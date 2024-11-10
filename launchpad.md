
## 相关文档&资料

https://documentation.ubuntu.com/launchpad/en/latest/

https://documentation.ubuntu.com/launchpad/en/latest/how-to/running/

https://dingjingmaster.github.io/2023/07/0014-launchpad%E6%90%AD%E5%BB%BA/

## 基础环境部署


官方推荐的环境为Ubuntu 20.04 LTS 和 Python 3.8。

**安装 snap**
```
sudo apt install snap
```
**安装 LXD**
```
sudo snap install lxd

lxd init 

```

**修改 lxd 镜像源**
```
lxc remote add tuna-images https://mirrors.tuna.tsinghua.edu.cn/lxc-images/ --protocol=simplestreams --public
```

**创建 LXD 容器**

```
#! /bin/sh

id=400000  # some large uid outside of typical range, and outside of already mapped ranges in /etc/sub{u,g}id
uid=$(id -u)
gid=$(id -g)
user=$(id -un)
group=$(id -gn)

# give lxc permission to map your user/group id through
sudo usermod --add-subuids ${uid}-${uid} --add-subgids ${gid}-${gid} root

# create a profile to control this
lxc profile create $user >/dev/null 2>&1

# configure profile
cat << EOF | lxc profile edit $user
name: $user
description: allow home dir mounting for $user
config:
  raw.idmap: |
    uid $uid $id
    gid $gid $id
  user.user-data: |
    #cloud-config
    runcmd:
      - "groupadd $group --gid $id"
      - "useradd $user --uid $id --gid $group --groups adm,sudo --shell /bin/bash"
      - "echo '$user ALL=(ALL) NOPASSWD:ALL' >/etc/sudoers.d/90-cloud-init-users"
      - "chmod 0440 /etc/sudoers.d/90-cloud-init-users"
devices:
  home:
    type: disk
    source: $HOME
    path: $HOME
EOF
```

执行上述脚本

**执行创建容器**

```
lxc launch ubuntu:20.04 lpdev -p default -p $USER
```
**配置容器访问**

使用 lxc info lpdev 查看容器IP

执行 ssh-keygen

将 ~/.ssh/id_rsa.pub 复制到 ~/.ssh/authorized_keys 中

执行：ssh -A $USER@你的容器IP 进入容器


配置账户：

```
mkdir ~/launchpad && cd ~/launchpad
```

修改配置文件 ~/.ssh/config

在这里注册账户：https://launchpad.net/+login

```
Host bazaar.launchpad.net
        User LPUSERNAME
Host git.launchpad.net
        User LPUSERNAME
```
继续执行下述命令：

```

mkdir ~/launchpad

cd ~/launchpad

curl https://git.launchpad.net/launchpad/plain/utilities/rocketfuel-setup >rocketfuel-setup

```
继续执行 

下述命令会执行该操作：

https://documentation.ubuntu.com/launchpad/en/latest/explanation/running-details/

```
chmod a+x rocketfuel-setup

./rocketfuel-setup
```

系统将提示您输入 sudo 密码和 Launchpad 登录账户

如果容器中已有 Apache 服务器，这将更改您的 Apache 配置。它还将修改 /etc/hosts ，并在容器中设置 PostgreSQL 。

至此，基础配置工作结束 下面是运行前设置

## 运行前设置

**请勿将 PostgreSQL 用于 Launchpad 以外的任何用途！！！**

设置PostgreSQL：
```
$ ./utilities/launchpad-database-setup $USER

make schema

```

## 运行 
```
make run
```

FAQ：见该页面的最后部分

https://documentation.ubuntu.com/launchpad/en/latest/how-to/running/
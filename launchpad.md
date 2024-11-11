
## 相关文档&资料

https://documentation.ubuntu.com/launchpad/en/latest/

https://documentation.ubuntu.com/launchpad/en/latest/how-to/running/

https://dingjingmaster.github.io/2023/07/0014-launchpad%E6%90%AD%E5%BB%BA/

## 部署（不使用容器）

此方法需要主机只运行Launchpad，同时相关组件版本对齐

系统 Ubuntu 20.04 Python3.8

克隆 Launchpad 源代码： 从 Launchpad 的 Git 仓库克隆代码：
```
git clone https://git.launchpad.net/launchpad
```
进入代码目录： 切换到克隆的代码目录中：

```
cd launchpad
```
运行安装脚本：

```
./utilities/rocketfuel-setup
```
数据库配置： 执行数据库初始化和配置，确保所有的服务和表都已正确设置：

```
make schema
```
启动 Launchpad

```
make run
```

## 部署（使用容器）

安装 LXD

```

sudo snap install lxd
sudo lxd init --auto
```

创建容器
```
lxc launch ubuntu:20.04 launchpad-container
```
设置网络

此处实例的采用的是桥接方式 接口为br0 仅供参考 请以实际部署为准

```
lxc network attach br0 launchpad-container eth0
```
把容器的接口 eth0 桥接到 br0 

安装依赖

```
lxc exec launchpad-container -- sudo apt update
lxc exec launchpad-container -- sudo apt install python3.8 postgresql rabbitmq-server git
```
下载Launchpad

在容器内克隆并设置 Launchpad：

```
lxc exec launchpad-container -- bash
git clone https://git.launchpad.net/launchpad
cd launchpad
./utilities/rocketfuel-setup
```
数据库初始化

```
make schema
```
启动 Launchpad

```
make run
```
通过下面的命令确定设备IP通过浏览器访问
```
lxc list
```


## 构建设备配置

基础环境安装
```
sudo apt update
sudo apt install sbuild ubuntu-dev-tools debootstrap
```

创建 sbuild 环境
```
sudo mk-sbuild --arch=riscv64 focal
```

配置 sbuild 环境

编辑 /etc/schroot/chroot.d/ 下的配置文件

```
[focal-riscv64-sbuild]
description=Ubuntu 20.04 riscv64 sbuild
directory=/srv/chroot/focal-riscv64-sbuild
root-users=root
type=directory
profile=sbuild
users=YOUR_USERNAME
```

配置 buildd

编辑 /etc/buildd/buildd.conf
```
BUILD_ARCH=riscv64
BUILD_CHROOT=riscv64-unstable
```

启动服务
```
sudo systemctl start buildd
```

在Launchpad的管理页面中进行连接

配置访问密钥，然后在构建机上导入
```
export LAUNCHPAD_BUILDER_API="http://your-launchpad-instance/api/devel/"
export LAUNCHPAD_BUILDER_KEY="/path/to/your/private/key"
```
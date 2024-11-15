
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
./utilities/launchpad-database-setup $USER
make schema
```
启动 Launchpad

```
make run
```

默认使用127.0.0.88服务，如需调整ip 请执行
```
sudo make LISTEN_ADDRESS='*' install
```
在访问的机器上按照如下配置hosts文件
```
# Launchpad virtual domains. This should be on one line.
<your container IPv4 address>     launchpad.test answers.launchpad.test archive.launchpad.test api.launchpad.test bazaar.launchpad.test bazaar-internal.launchpad.test blueprints.launchpad.test bugs.launchpad.test code.launchpad.test feeds.launchpad.test keyserver.launchpad.test lists.launchpad.test ppa.launchpad.test private-ppa.launchpad.test testopenid.test translations.launchpad.test xmlrpc-private.launchpad.test xmlrpc.launchpad.test
```



使用 admin@canonical.com 账号在 launchpad.test 登录

向数据库中添加riscv构建机参数 无需重启 修改完毕即生效

```
\c launchpad_dev  

进入 launchpad_dev 数据表

INSERT INTO public.processor (id, name, title, description, restricted, build_by_default, supports_nonvirtualized, supports_virtualized) VALUES (4, 'riscv
64', 'RISC-V 64bit', 'RISC-V 64bit', false, true, true, true);
```

在 https://launchpad.test/builders/+new 添加构建机器


Open resources: (Optional)  此处可写入构建机器的Tag

Resource tags offered by this builder, that can be required by a build and if required must match.




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
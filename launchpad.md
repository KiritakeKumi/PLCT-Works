
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


获取程序buildd

$ sudo apt install git
$ git clone https://git.launchpad.net/launchpad-buildd

安装依赖项

$ cd launchpad-buildd
$ sudo apt-add-repository ppa:launchpad/ubuntu/buildd-staging
$ sudo apt-add-repository ppa:launchpad/ubuntu/ppa
$ vi /etc/apt/sources.list.d/launchpad-ubuntu-ppa-bionic.list <uncomment deb-src line>
$ sudo apt update
$ sudo apt build-dep launchpad-buildd fakeroot
$ sudo apt install -f

注意：如果fakeroot找不到请尝试：

$ sudo sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt build-dep launchpad-buildd fakeroot
$ sudo apt install -f

制作软件包

$ cd launchpad-buildd
$ make
$ cd ..
$ sudo dpkg -i ./python3-lpbuildd_<version>_all.deb ./launchpad-buildd_<version>_all.deb

$ git clone https://git.launchpad.net/ubuntu-archive-tools
$ sudo apt install python3-launchpadlib python3-ubuntutools

下载 chroot 镜像：

./manage-chroot -s jammy -a riscv64 get
sha1sum livecd.ubuntu-base.rootfs.tar.gz
mv livecd.ubuntu-base.rootfs.tar.gz <sha1sum from previous line>
sudo cp <sha1sum named file> /home/buildd/filecache-default
sudo chown buildd: /home/buildd/filecache-default/<sha1sum named file>

如果需要修改git相关位置可在launchpad/launchpad/configs/development/launchpad-lazr.conf 和 launchpad/launchpad/lib/lp/services/config/schema-lazr.conf
修改git_browse_root、git_ssh_root等项目

运行额外的服务（launchpad）

utilities/start-dev-soyuz.sh
utilities/soyuz-sampledata-setup.py
make run

在 https://launchpad.test/builders 上注册构建机器

机器地址类似于 http://<buildd ip>:8221

提交完毕后 大约30秒会在网页中显示为空闲状态，即连接成功
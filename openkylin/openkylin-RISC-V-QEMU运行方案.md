


# openkylin-RISC-V-QEMU运行方案

## 前言 

本文档记录的是使用visionfive包在QEMU上运行。


## 开始动手

由于visionfive包本身就使用的u-boot方式启动 所以此处就将计就计


1. 到https://toolchains.bootlin.com/ 下载riscv64工具链，将bin目录添加到PATH

```
    wget https://toolchains.bootlin.com/downloads/releases/toolchains/riscv64-lp64d/tarballs/riscv64-lp64d--glibc--stable-2021.11-1.tar.bz2

    tar -jxvf riscv64-lp64d--glibc--stable-2021.11-1.tar.bz2

    export TOOLPATH=$(pwd)/riscv64-lp64d--glibc--stable-2021.11-1

    export PATH=$PATH:$TOOLPATH/bin
```   


2. 构建u-boot

```
    git clone https://gitlab.denx.de/u-boot/u-boot.git

    cd u-boot

    apt install flex

    apt install libssl-dev

    make CROSS_COMPILE=riscv64-linux- qemu-riscv64_smode_defconfig

    make CROSS_COMPILE=riscv64-linux- -j $(nproc)
```


3. 移除现有镜像中boot分区的dtb描述文件

```
此处只需单独挂载img中boot分区 并删除extlinux.conf文件中包含dtb文件的所在行
```

4. 将相关文件检查完整移入同一个目录

```
一共是以下两个文件

u-boot.bin  此文件即第二步编译的结果

openkylin-0.7.5-visionfive-riscv64.img  经过第三步修改的官方img文件

```


5. 启动QEMU

```
qemu-system-riscv64 \
-machine virt -nographic -m 4096 -smp 12 \
-kernel u-boot.bin \
-device virtio-net-device,netdev=eth0 -netdev user,id=eth0 \
-drive file=openkylin-0.7.5-visionfive-riscv64.img,format=raw,if=virtio
-object rng-random,filename=/dev/urandom,id=rng0 \
-device virtio-rng-device,rng=rng0 \
  -device virtio-blk-device,drive=hd0 \
  -device virtio-net-device,netdev=usernet \
  -netdev user,id=usernet,hostfwd=tcp::12055-:22 \
  -append 'root=/dev/vda1 rw console=ttyS0 systemd.default_timeout_start_sec=600 selinux=0 highres=off mem=4096M earlycon' \
  -bios none

  ```
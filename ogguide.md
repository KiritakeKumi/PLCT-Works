# OpenGauss RISC-V 基础使用指南

最后修订日期：

## 简介

RISC-V是一个开源的精简指令集架构（ISA），基于RISC（精简指令集计算）理念，旨在提供一个简洁、高效、可扩展的指令集。它由加利福尼亚大学伯克利分校于2010年发布，并迅速在学术界和工业界获得广泛关注。RISC-V支持多种位宽和可选的扩展指令，如浮点、向量和加密指令，具备高度的可定制性。作为开源架构，RISC-V允许开发者自由实现并根据需求扩展，是嵌入式系统、移动设备、高性能计算等领域的理想选择。

目前 OpenGauss RISC-V SIG 在由算能、RISC-V 中国社区、中科院软件所PLCT实验室联合发起举办的 “RISC-V 软件移植优化锦标赛”成果基础上在SG2042平台可以运行OpenGauss。


## 开始上手

我们目前推荐在 openEuler riscv64 上进行构建和使用,此处指南使用SG2042作为基础平台

**进入目录**
cd /root/rpmbuild/SOURCES

**安装必要工具**
dnf install -y rpm-build rpmdevtools dnf-plugins-core

**安装编译依赖**
yum-builddep -y opengauss-server.spec

**下载源码**
spectool -g opengauss-server.spec

**开始编译**
rpmbuild -ba opengauss-server.spec

**安装编译完成的RPM包**
dnf install -y opengauss-5.1.0.rpm

**启动服务**
systemctl enable --now opengauss-server
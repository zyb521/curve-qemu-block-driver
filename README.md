# Curve Support for Qemu

## 概述

[CURVE](https://github.com/opencurve/curve)是网易自主设计研发的高性能、高可用、高可靠分布式存储系统，具有非常良好的扩展性。当前基于CURVE已经实现了高性能块存储系统，支持快照克隆和恢复，支持QEMU虚拟机和物理机NBD设备两种挂载方式，在网易内部作为高性能云盘使用。

NBD挂载方式已经在Curve代码中提供，当前仓库提供了Qemu挂载Curve块设备的支持。

Qemu挂载Curve块设备提供了两种方式：通过Curve-Client方式挂载、通过NEBD挂载。两种方式的区别可以参考[NEBD文档](https://github.com/opencurve/curve/blob/master/docs/cn/nebd.md)。

**注意事项：**

使用之前需要部署一个Curve集群，以及创建一个Curve卷，具体步骤参考[使用 CurveAdm 部署 CurveBS 集群](https://github.com/opencurve/curveadm/wiki/curvebs-cluster-deployment)。

目前提供了基于 QEMU v2.8.0 v4.2.0 v5.2.0 v6.2.0 版本的 patch.

## 使用方式（以 QEMU v2.8.0 版本为例）

### 通过Curve-Client挂载

1. 安装Curve-SDK，参考[Curve手动部署sdk](https://github.com/opencurve/curve/blob/master/docs/cn/curve%E9%80%9A%E8%BF%87tar%E5%8C%85%E6%89%8B%E5%8A%A8%E9%83%A8%E7%BD%B2sdk.md)

2. 修改`/etc/curve/client.conf`中`mds.listen.addr`配置项的值，指向Curve集群的MDS地址，多个地址用`,`分隔

3. 将`curve-client-qemu-v2.8.0.patch`应用到Qemu v2.8.0版本的代码上，然后编译

   参考步骤：

   ```bash
   git clone https://github.com/opencurve/curve-qemu-block-driver
   git clone https://github.com/qemu/qemu.git
   cd qemu
   git checkout v2.8.0
   patch -p1 < ../qemu-block-driver/qemu/curve-client-qemu-v2.8.0.patch
   mkdir build && cd build
   ../configure --target-list=x86_64-softmmu
   make -j`getconf _NPROCESSORS_ONLN`
   ```

4. 启动Qemu，并添加挂载Curve盘参数，例如：

   ```bash
   ./x86_64-softmmu/qemu-system-x86_64 -L pc-bios/ \
       -smp 8 \
       -m 8192 \
       -drive format=raw,file=/root/Debian_7_x86_64-flat.vmdk.img,cache=none,if=virtio \
       -drive format=raw,file=cbd:pool//qemu0_qemu_:/etc/curve/client.conf,if=virtio \
       -boot c \
       -net nic,model=virtio,macaddr=66:66:66:66:66:0a -net tap,ifname=brostub,script=no \
       -rtc base=localtime \
       -vnc :3 \
       -enable-kvm
   ```

   具体启动参数比如系统镜像路径、网卡等需要根据不同配置进行修改，**挂载Curve盘相关参数在第5行**，file中的参数格式为`cbd:存储池/卷名_用户名_[密码]:client配置文件路径`，对应上述参数：

   - cbd：表示挂载Curve卷
   - pool：Curve卷所属的存储池，当前未使用
   - /qemu0_curve_：/qemu0为提前创建的Curve卷，qemu为卷所属的用户，密码可选，如果创建卷时未指定，可以省略
   - /etc/curve/client.conf：client配置文件路径

5. 进入Qemu虚拟机，并进行fio测试

### 通过NEBD挂载

（待补充）

## libvirt支持

libvirt 目录下的 patch 提供了对libvirt的支持，目前支持了 v2.4.0 v5.10.0 v6.10.0 v7.10.0，其他版本可能需要相应的修改


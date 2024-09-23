# ramdisk

## 一、什么是ramdisk

### 1 Ramdisk 介绍与分类

在 Linux 系统中，Ramdisk 分为两种：

1. **Initrd (Initial RAM Disk)**  
   - **支持版本**：从 Linux 内核 2.0/2.2 开始支持。
   - **特点**：可以格式化并加载，但其大小固定，灵活性较差。

2. **Initramfs (Initial RAM Filesystem)**  
   - **支持版本**：从 Linux 内核 2.4 开始支持。
   - **特点**：通过 `ramfs` 实现，无法被格式化，但使用方便。其大小会根据所需空间动态调整，是当前 Linux 系统中常用的 Ramdisk 技术。

如果你想了解更多关于 initrd 和 initramfs 的区别，可以参考博主的另一篇文章《initramfs的介绍与制作》。

### 2 Ramdisk 的作用与优势

Ramdisk 实际上是从内存中划出一部分作为一个分区使用，换句话说，就是将内存的一部分当作硬盘来使用，可以在其中存储文件。Ramdisk 并非一个实际的文件系统，而是一种将实际的文件系统装入内存的机制，且可以作为根文件系统。它使用的文件系统通常是 `ext2`。

**使用 Ramdisk 的原因：**
1. **性能提升**：
   - 如果某些文件需要频繁使用，将它们加载到内存中可以显著提高程序的运行速度，因为内存的读写速度远高于硬盘。
   - 由于内存价格相对低廉，如今一台 PC 拥有 128MB 或 256MB 内存已很常见。划出部分内存来提升整体性能的效果，不亚于更换新的 CPU。

2. **系统恢复**：
   - Ramdisk 是基于内存的文件系统，具有断电后不保存数据的特性。如果对文件系统进行操作导致系统崩溃，只需重新上电即可恢复系统，无需担心数据的持久性问题。

Ramdisk 通过将部分内存用作存储介质，可以显著提升系统性能，并提供一种易于恢复的系统运行环境。这种技术尤其适用于需要高性能和安全性（易于恢复）的应用场景。

## 二、ramdisk的制作

以下是对该段资料的整理：

### 1 建立根文件系统

**（1）创建根文件系统目录**
- 创建目录结构：
  ```bash
  mkdir rootfs  
  cd rootfs
  mkdir root dev etc boot tmp var sys proc lib mnt home usr   
  mkdir etc/init.d etc/rc.d etc/sysconfig  
  mkdir usr/sbin usr/bin usr/lib usr/modules  
  mkdir var/lib var/lock var/run var/tmp
  sudo mknod -m 600 dev/console c 5 1  
  sudo mknod -m 600 dev/null  c 1 3
  ```
- 建议将以上命令写成脚本，避免手动输入。

**（2）拷贝交叉编译工具中的库**

- 找到交叉编译工具安装目录，使用`which`命令：
  ```bash
  which your-toolchain-gcc
  ```
- 拷贝动态库到lib目录：
  ```bash
  cp /path/to/cross-compiler/lib/* /home/mkrootfs/rootfs/lib -rfd
  ```

**（3）建立`etc`目录下的配置文件**
- 拷贝主机`etc`目录下的`passwd`、`group`、`shadow`文件到`rootfs/etc`目录。
- 创建`etc/mdev.conf`（内容为空）。
- 在`etc/sysconfig`目录下新建`HOSTNAME`文件，内容为“jimmy”。
- 编辑`etc/inittab`文件：
  ```bash
  ::sysinit:/etc/init.d/rcS
  console::askfirst:-/bin/sh
  ::restart:/sbin/init  
  ::ctrlaltdel:/sbin/reboot     
  ::shutdown:/bin/umount -a -r  
  ::shutdown:/sbin/swapoff -a  
  ```
- 创建`etc/init.d/rcS`文件，并修改权限：
  ```bash
  #!/bin/sh
  PATH=/sbin:/bin:/usr/sbin:/usr/bin
  runlevel=S
  prevlevel=N
  umask 022
  export PATH runlevel prevlevel
  echo "----------mount all----------------"
  mount -a
  echo "****************Hello jimmy*********************"
  echo "Kernel version:linux-3.18.28"
  echo "***********************************************"
  /bin/hostname -F /etc/sysconfig/HOSTNAME
  ```
  ```bash
  chmod +x /etc/init.d/rcS
  ```
- 编辑`etc/fstab`文件：
  ```bash
  proc      /proc           proc       defaults     0         0  
  none      /tmp            ramfs      defaults     0         0  
  sysfs     /sys            sysfs      defaults     0         0
  ```
- 编辑`etc/profile`文件：
  ```bash
  USER="id -un"
  LOGNAME=$USER
  PS1='[\u@\h $PWD]#'
  PATH=$PATH
  HOSTNAME='/bin/hostname'
  export USER LOGNAME PS1 PATH
  echo “-----/etc/profile-------”
  ```

**（4）交叉编译BusyBox**
- 从BusyBox官网下载源码。
- 修改源码根目录下的`Makefile`：
  ```bash
  CROSS_COMPILE ?=arm-none-linux-gnueabi-
  ARCH ?=arm
  ```
- 编译BusyBox：
  ```bash
  make distclean
  make defconfig
  make
  make CONFIG_PREFIX=/home/mkrootfs/rootfs install
  ```

**（5）编译安装内核模块**
  ```bash
  make modules ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi-
  make modules_install INSTALL_MOD_PATH=/home/mkrootfs/rootfs
  ```

**（6）使用工具制作ramdisk文件系统**
- 下载genext2fs工具，制作ramdisk文件系统：
  ```bash
  genext2fs -b 4096 -d rootfs ramdisk
  gzip -9 -f ramdisk
  ```

### 2 修改已有文件系统

**（1）解压**
  ```bash
  gunzip ramdisk.gz
  ```

**（2）挂载镜像文件**
  ```bash
  mkdir /mnt/loop
  mount –o loop ramdisk /mnt/loop
  ```

**（3）对文件系统进行操作**
  ```bash
  cd /mnt/loop
  ```

**（4）卸载文件系统**
  ```bash
  umount /mnt/loop
  ```

**（5）压缩文件系统**
  ```bash
  gzip –v9 ramdisk
  ```
# 在SSD存储器（U盘或硬盘）上安装Ubuntu

- 目标：实现Ubuntu系统的即插即用，在SSD存储器（U盘或硬盘）上安装Ubuntu，在任意电脑的USB接口（defult：USB3.1）上插入存储器，即可通过选择BOOT方式（开机时按F12）启动Ubuntu

- 以下是本文的安装环境：

  - SSD存储器：thinkplus TU180 PRO 512G（空间原则上大于64G）

  - 电脑型号：huawei matebook 14 2020版

  - 处理器：Intel® Core™ i7-10510U CPU @ 1.80GHz × 8

  - 内存：16G
  
  - ubuntu版本：Ubuntu22.04.1TLS 64位
  - 电脑USB版本：USB3.1
  - SSD存储器USB版本：USB3.2
- 如果你的电脑硬盘使用了特殊的读写协议可能导致安装失败，已知外星人笔记本有此问题。该特性不影响系统的正常启动，即安装好的ubuntu可以正常启动，只是在这样的电脑上无法正常进行安装流程。可以考虑使用其他电脑在移动存储器上安装系统

## 一、下载Ubuntu22.04系统的iso文件

- https://cn.ubuntu.com/download/desktop

## 二、制作U盘启动系统

- 需要windows系统下操作

- 下载地址https://github.com/pbatard/rufus/releases

- 参考https://ubuntu.com/tutorials/create-a-usb-stick-on-windows，不赘述
- 也可以使用[balenaEtcher](https://www.balena.io/etcher/)制作引导盘

## 三、安装ubuntu系统

1. 将两个U盘插上电脑，开启电脑。按F12进入BIOS（从开机一直按就行）

    这一步可能因电脑不同而不同，详细信息自行搜索

2. 选择引导盘的UEFI BOOT回车，一般来说该BOOT和引导盘的名字相关

    如：使用金士顿的U盘做引导盘，则对应BOOT一般以Kingston（不区分大小写）开头

3. 以上流程可参考https://ubuntu.com/tutorials/install-ubuntu-desktop

4. **想要在SSD存储器上安装Ubuntu，[上述教程](https://ubuntu.com/tutorials/install-ubuntu-desktop)的6、7步需要作出调整**

    - 在**安装类型（Install type）**页面选择**其他选项（Something else）**，点击下一步

        ![](https://images.cnblogs.com/cnblogs_com/blogs/743211/galleries/2294100/o_de611426.png)

    - 进入**分区界面**。sda 一般是本地电脑，代表的是你自己电脑上的硬盘。我选中我自己空闲的另一个U盘，一般是以sde或sdc开头。选中此空闲选项，单击“+”。（若磁盘中有内容，可先新建分区列表）

        ①：设置EFI引导区（这个分区一定要是存储器的第一个分区）
        因为是U盘的UEFI启动，因此需要设置EFI引导。系统引导文件都在里面。
        大小：大小最好不要小于256MB，这里给512MB。
        ②：swap虚拟内存区
        swap交换空间，也就是虚拟内存的地方。
        大小：根据机器的实际内存大小来定，一般和电脑内存一样即可，我的电脑内存16G，给16G即可考虑到以后可能更换内存更大的电脑，32G也无不可。
        ③：挂载点：/usr
        存放用户程序，一般在/usr/bin中存放发行版提供的程序，用户自行安装的程序默认安装到/usr/local/bin中。就像是Windows里面的Program Files.
        用于：Ext4日志文件系统。
        大小：这个必须要大，512G的存储可以给到200G。

        备注：此分区可以不单独挂载

        ④：挂载点：/home
        主要用来存放用户数据。这个分区的大小取决于有多少用户。类似于Windows里的User。图片、视频、下载内容基本都在里面，不像在Win上面你想移动就移动。
        用于：Ext4日志文件系统
        大小：根据需求分配，512G的存储可以给到200G。

        备注：理论上也可以不单独挂载，但这个分区相当于windows的D盘，用于存储用户文件，建议单独挂载

        ⑤：挂载点：/
        主要用来存放系统文件。“/”把除了你之前挂载的home和usr外的全部杂项囊括，分配空间不要太小。
        用于：Ext4日志文件系统
        大小：至少5G以上，简单来说就是剩下多少空间就分多少。

    - 安装启动引导器的设备选择你的SSD存储器，点击**现在安装**，会提示分区的变更和格式化的信息，直接确认即可

5. 按照[上述教程](https://ubuntu.com/tutorials/install-ubuntu-desktop)从第八步开始继续安装即可

6. 安装完成后，建议**不要**点击**立即重启**，可以关闭弹窗，具体原因请继续阅读下文

## 四、boot-repair（重要）

按照上述过程安装系统后，根据提示拔掉U盘，能够正常进入Ubuntu系统。

但是，当把移动固态硬盘插到其他电脑时，是不能正常使用的，原因是存储设备的ESP分区中没有引导文件，解决过程如下：

1. 重新插上引导盘和移动固态硬盘，进入BIOS选择引导盘启动，进入引导盘中的Ubuntu系统后，选择 Try Ubuntu without installing

2. 连上网络，安装 **boot-repair** 软件：

    ```shell
    sudo apt-add-repository ppa:yannubuntu/boot-repair
    sudo apt update
    sudo apt install boot-repair
    ```

3. 安装完成后，打开终端执行该软件

    ```shell
    # 打开一个终端
    boot-repair
    ```

4. 选择 **“Recommended repair（推荐修复）”** ，等待修复完成

5. 确认修复完成：

    > 如果修复完成，在ESP分区会出现一个名为 EFI 的目录，里面有 BOOT 和 ubuntu 两个子目录，用来启动 Ubuntu 系统的引导文件就是位于 ubuntu 目录中的 shimx64.efi 文件

6. 确认修复完成后，就可以将移动固态硬盘插在不同电脑上运行Ubuntu系统了。

## 五、要点总结

- 注意分区时EFI引导区一定要在**第一个**分区上
- 一定要进行**boot-repair**
- 如果你的存储设备无法被正常读取尝试通过DiskGenius（或类似功能的软件）将磁盘类型转换为GUID格式。

## 六、参考教程

- [在U盘下安装ubuntu20.04，从U盘启动Linux系统](https://blog.csdn.net/qq_51491920/article/details/123668279)
- [Ubuntu系统安装在移动固态硬盘，实现在不同电脑即插即用](https://blog.csdn.net/hypc9709/article/details/127941834?spm=1001.2014.3001.5506)
- [移动硬盘安装Ubuntu系统（UEFI引导）的一些记录](https://blog.csdn.net/u012939909/article/details/80753115)


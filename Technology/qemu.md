# QEMU

>   QEMU 是一个支持跨平台虚拟化的虚拟机，有 user mode 和 system mode 两种配置方式。其中 qemu 在 system mode 配置下模拟出整个计算机，可以在 qemu 之上运行一个操作系统。VMware 和 Virtualbox 之类通常只能在 x86 计算机上虚拟出一个 x86 虚拟机，而 QEMU 支持在 x86 上虚拟出一个 ARM 虚拟机。qemu 在 user mode 配置下，可以运行跟当前平台指令集不同的平台的可执行程序。
>
>   QEMU 中有两个重要的名词 host 和 target （guest），其中 host 表示 qemu 程序本身运行的平台，target（guest）表示 qemu 虚拟出的计算平台（system mode）或支持的可执行程序的运行平台（user mode）。

## 〇、环境

-   架构：x86_64

-   操作系统：Ubuntu 22.04.2 LTS（64位）
-   gcc version 11.4.0 

## 一、编译

### 1 安装依赖

#### 1.1 Ninja

Ninja是一个构建系统，专为速度而设计，通常用于加速软件项目的构建过程。你可以使用系统的包管理器安装Ninja，或者从源代码构建和安装。

##### 1.1.1 使用包管理器安装Ninja：

Ubuntu/Debian:

```
sudo apt-get install ninja-build
```

Fedora:

```
sudo dnf install ninja-build
```

CentOS:

```
sudo yum install ninja-build
```

Arch Linux:

```
sudo pacman -S ninja
```

##### 1.1.2 从源代码构建和安装Ninja：

1.  获取源代码：

    ```
    git clone git://github.com/ninja-build/ninja.git
    ```

2.  进入源代码目录：

    ```
    cd ninja
    ```

3.  配置并构建：

    ```
    ./configure.py --bootstrap
    ```

    这将生成可执行文件 `ninja`。

4.  将`ninja`可执行文件复制到系统路径中：

    ```
    sudo cp ninja /usr/local/bin/
    ```

    或者你可以选择将可执行文件放在其他目录，并确保该目录在你的`$PATH`环境变量中。

验证Ninja是否已成功安装：

```
ninja --version
```

如果一切顺利，你应该看到Ninja的版本信息。

#### 1.2 glib-2.56 gthread-2.0

安装GLib 2.56和GThread 2.0库的具体步骤取决于使用的操作系统。以下是一些常见的操作系统的安装命令：

Ubuntu/Debian:

```bash
sudo apt-get install libglib2.0-dev
```

Fedora:

```bash
sudo dnf install glib2-devel
```

CentOS:

```bash
sudo yum install glib2-devel
```

Arch Linux:

```bash
sudo pacman -S glib2
```

macOS (使用Homebrew):

```bash
brew install glib
```

这些命令会安装GLib的开发包，其中包括用于编译和链接的头文件和库文件。GLib通常包含了GThread，所以不需要单独安装GThread。

#### 1.3 Meson

要安装 Meson 构建系统，你可以使用系统的包管理器或通过 Python 的 pip 工具安装。以下是两种常见的安装方法：

##### 1.3.1 通过包管理器安装 Meson：

Ubuntu/Debian:

```bash
sudo apt-get install meson
```

Fedora:

```bash
sudo dnf install meson
```

CentOS:

```bash
sudo yum install meson
```

Arch Linux:

```bash
sudo pacman -S meson
```

##### 1.3.2 通过 Python pip 安装 Meson：

如果你的系统中已经安装了 Python 和 pip，你可以使用以下命令安装 Meson：

```bash
pip install meson
```

请注意，有些系统可能需要使用 `pip3` 而不是 `pip`：

```bash
pip3 install meson
```

安装完成后，你可以验证 Meson 是否成功安装，运行以下命令：

```bash
meson --version
```

这应该输出安装的 Meson 版本信息，确认安装成功。

#### 1.4 Pixmen

Pixman 是一个用于像素操作的图形库，通常被其他图形库如 Cairo 使用。你可以使用系统的包管理器安装 Pixman。以下是一些常见系统的安装命令：

Ubuntu/Debian:

```bash
sudo apt-get install libpixman-1-dev
```

Fedora:

```bash
sudo dnf install pixman-devel
```

CentOS:

```bash
sudo yum install pixman-devel
```

Arch Linux:

```bash
sudo pacman -S pixman
```

macOS (使用Homebrew):

```bash
brew install pixman
```

这些命令将安装 Pixman 的开发包，其中包括用于编译和链接的头文件和库文件。

### 2 编译

以编译龙芯la32r的qemu为例：

1.   拉取代码

     ```bash
     # 官方
     git clone https://github.com/qemu/qemu.git
     # 支持la32r指令集
     git clone --recursive git@gitee.com:loongson-edu/la32r-QEMU.git
     ```

2.   编译

     ```bash
     cd qemu
     mkdir build
     cd build
     ../configure --target-list=loongarch32-softmmu --disable-werror --enable-debug
     make -j16  # 可根据实际平台调整编译所用线程数量
     make install  # 可以省略（有用的编译结果只有qemu-system-loongarch32这个可执行文件）
     ```

     更多的编译选项可通过`../configure --help`查看

3.   测试

     命令行运行

     ```bash
     ./qemu-system-loongarch32 --version
     ```

     有如下输出

     `````
     QEMU emulator version 6.2.50
     Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
     `````

## 二、System mode使用


# WSL安装

- huawei win10 22H2
- WSL2
- 通常建议使用 `wsl --install`安装适用于 Linux 的 Windows 子系统（Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 ），但如果运行的是旧版 Windows，则可能不支持这种方式

## 一、手动安装

- 启用适用于 Linux 的 Windows 子系统

  ```shell
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```

- 启用虚拟机功能

  ```shell
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```

- 下载 Linux 内核更新包

  - 下载最新包

    ```http
    https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
    ```

  - 运行上一步中下载的更新包。（双击以运行 - 系统将提示你提供提升的权限，选择“是”以批准此安装。）

- 将 WSL 2 设置为默认版本

  ```shell
  wsl --set-default-version 2
  ```

- 安装所选的 Linux 分发
  - 打开 Microsoft Store，并选择你偏好的 Linux 分发版。
  - 在分发版的页面中，选择“获取”
  - 首次启动新安装的 Linux 分发版时，将打开一个控制台窗口，系统会要求你等待一分钟或两分钟，以便文件解压缩并存储到电脑上。 未来的所有启动时间应不到一秒。
  - 然后，需要为新的 Linux 分发版创建用户帐户和密码

## 二、迁移Linux的位置（if you want）

- 停止WSL子系统

  ```shell
  wsl --shutdown
  ```

- 查看wsl状态

  ```shell
  wsl -l -v
  ```

  ```shell
  C:\Users\xxx\Desktop>wsl -l -v
    NAME      STATE           VERSION
  * Ubuntu    Stopped         2
  ```

  查看wsl下的Linux是否为关闭状态，当STATE为Stopped才能进行下一步

- 导出系统镜像

  ```shell
  wsl --export Ubuntu D:\UbuntuWSL\ubuntu.tar
  ```

- 注销原有的linux系统

  ```shell
  wsl --unregister Ubuntu
  ```

- 导入系统

  ```shell
  wsl --import <导入的Linux名称> <导入盘的路径> <ubuntu.tar的路径> --version 2 (代表wsl2)
  ```

  ```shell
  wsl --import Ubuntu D:\UbuntuWSL\ D:\UbuntuWSL\ubuntu.tar --version 2
  ```

- 修改默认用户

  打开wsl ubuntu之后，默认以root身份登录

  ```shell
  ubuntu.exe config --default-user <user name>
  ```

  - 在导入任意盘linux系统时，我起名Ubuntu，所以这里是ubuntu.exe；

    如果你起的名字是Ubuntu-20.04，那这里就是ubuntu2004.exe；

    如果你起的名字是ubuntu-18.04，那这里就是ubuntu1804.exe

  - \<user name>是原有wsl ubuntu的用户名称

## 三、配置代理

- Add "allow" rule to Windows firewall for WSL2 network

  ```bat
  New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
  ```

- 在`~/.bashrc`中添加WSL代理设置

  ```bash
  host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
  # 上面的不行就用下面这个
  # host_ip=$(ip route | grep default | awk '{print $3}')
  export ALL_PROXY="http://$host_ip:7890"
  # export HTTP_PROXY=$ALL_PROXY
  # export http_proxy=$ALL_PROXY
  # export HTTPS_PROXY=$ALL_PROXY
  # export https_proxy=$ALL_PROXY
  ```

- 在WIN代理软件中允许局域网连接

## 四、WSLG

### 1 中文显示

-   生成locale配置文件

    ```bash
    sudo locale-gen
    locale
    ```

-   Sharing Windows fonts with WSL

    ```bash
    sudo apt install fontconfig
    sudo vim /etc/fonts/local.conf
    ```

-   paste

    ```html
    <?xml version="1.0"?>
    <!DOCTYPE fontconfig SYSTEM "fonts.dtd">
    <fontconfig>
        <dir>/mnt/c/Windows/Fonts</dir>
    </fontconfig>
    ```

-   重启终端
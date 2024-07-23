# VCS

## 〇、环境

-   ubuntu 22.04.2 LTS
-   vivado2019.2
-   vcs_mx2018 

## 一、安装

### 1. 下载

- TODO

### 2.  安装

-   TODO

## 二、license

-   TODO

## 三、遇到过的问题和解决方案

### 1. (lmgrd) Failed to open the TCP port number in the license.

-   先确定环境中的LM_LICENSE_FILE信息配置正确

-   如果问题仍然存在，则是由于27000端口被占用

    -   方法一，尝试使用`lmdown`结束lmgrd进程，释放27000端口

    -   方法二

        ```shell
        sudo netstat -pan | grep 27000 # 查看占用27000端口的进程
        kill -9 xxx # 杀死有关进程
        ```

### 2.  /usr/bin/ld: rmapats_mop.o: .symtab local symbol at index 370 (>= sh_info of 1)

-   上面index后的紧跟的数字可能有不一样

-   出现这个问题很可能是因为vivado和vcs的版本不匹配，需要更换版本

    参阅https://support.xilinx.com/s/article/68324

-   还有可能因为gcc 4.8没有安装好

### 3. g++-4.8: error: unrecognized command line option '-no-pie'^[1]^

-   出现该错误的原因是g++版本较低，VCS的预编译选项中会添加-no-pie标志，但g++不能够识别该命令，在VCS编译完成后的/csrc/Makefile文件中，会增加预编译选项

-   解决方法：修改VCS bin目录下的vcs运行文件

    ```sh
    if [ $v = 1 ];  then
        SNPS_VCS_INTERNAL_UBUNTU_PRE_LDFLAGS=" -no-pie " 
        export SNPS_VCS_INTERNAL_UBUNTU_PRE_LDFLAGS ;
    fi			
    ```

    将以上内容的`-no-pie`去掉即可

    ```sh
    if [ $v = 1 ];  then
        SNPS_VCS_INTERNAL_UBUNTU_PRE_LDFLAGS="" 
        export SNPS_VCS_INTERNAL_UBUNTU_PRE_LDFLAGS ;
    fi
    ```

### 4. gcc-4.8和g++4.8安装方法^[2]^

-   由于系统版本更新直接安装一般来说无法成功

-   gcc-4.8 和 g++-4.8 软件包已在 Ubuntu 20.04 及更高版本的默认存储库中停止使用，但它们在 Ubuntu 18.04 默认存储库中仍然可用

-   使用以下命令安装

    ```shell
    cd Desktop/
    mkdir install_g++-4.8
    cd install_g++-4.8/
    sudo apt update
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/g++-4.8_4.8.5-4ubuntu8_amd64.deb 
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/libstdc++-4.8-dev_4.8.5-4ubuntu8_amd64.deb 
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/gcc-4.8-base_4.8.5-4ubuntu8_amd64.deb 
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/gcc-4.8_4.8.5-4ubuntu8_amd64.deb 
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/libgcc-4.8-dev_4.8.5-4ubuntu8_amd64.deb 
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/cpp-4.8_4.8.5-4ubuntu8_amd64.deb 
    wget http://mirrors.kernel.org/ubuntu/pool/universe/g/gcc-4.8/libasan0_4.8.5-4ubuntu8_amd64.deb  
    sudo apt install ./gcc-4.8_4.8.5-4ubuntu8_amd64.deb ./gcc-4.8-base_4.8.5-4ubuntu8_amd64.deb ./libstdc++-4.8-dev_4.8.5-4ubuntu8_amd64.deb ./cpp-4.8_4.8.5-4ubuntu8_amd64.deb ./libgcc-4.8-dev_4.8.5-4ubuntu8_amd64.deb ./libasan0_4.8.5-4ubuntu8_amd64.deb ./g++-4.8_4.8.5-4ubuntu8_amd64.deb
    ```


### 5. undefined reference to 'pthread_yield'^[3]^

-   原因是：vcs_mx_vO-2018.09-SP2/linux64/lib/vcs_save_restore_new.o 中的pthread_yield已在最新的glibc放弃使用了。

-   解决办法：

    -   备份

        ```shell
        cd vcs_mx/vO-2018.09-SP2/linux64/lib
        mv vcs_save_restore_new.o vcs_save_restore_new.o.bak
        ```

    -   替换符号表中的pthread_yield

        ```shell
        objcopy --redefine-sym pthread_yield=sched_yield ./vcs_save_restore_new.o.bak ./vcs_save_restore_new.o
        ```

### 6. libvcsnew.so: undefined reference to xxx^[4]^

-   在利用vcs进行仿真的时候，遇到如下 “libvcsnew.so: undefined reference to xxxxxx" 报错

    ```tcl
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `vfs::vfsReleaseVirtualFileList(VFILE_LIST*)'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `ZsGetDfsInstId'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `snpsCheckCallocFromGroupFunc'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `ZsFinalize'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `vfs_feof'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `snps_mem_named_push_context'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `vfs_fflush'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `vfs_fseek'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `snpsGroupSize'
    /opt/synopsys/vcs_vO-2018.09-SP2/linux64/lib/libvcsnew.so: undefined reference to `vfs_enable_multi_process_support'
    ...
    ```

-   在vcs命令后面加入选项

    ```shell
    -full64 -cpp g++-4.8 -cc gcc-4.8 -LDFLAGS -Wl,--no-as-needed
    ```

    如果在vivado中调用vcs，则在Tools>Settings>Project Settings>Simulation>Elaboration>vcs.elaborate.vcs.more_options中加入以上选项

## 四、参考文献

-   \[1][Ubuntu虚拟机中VCS安装和启动过程中遇到的问题总结](https://blog.csdn.net/heyuming20062007/article/details/126852329)
-   \[2][Need gcc and g++ 4.8 in Ubuntu 22.04.1](https://askubuntu.com/questions/1450426/need-gcc-and-g-4-8-in-ubuntu-22-04-1)
-   \[3][求助Ubuntu 22.04 中 VCS 2018.09-SP2遇到undefined reference to `pthread_yield\'的错误](https://bbs.eetop.cn/thread-928873-1-1.html)
-   \[4][libvcsnew.so: undefined reference to](https://blog.csdn.net/hh199203/article/details/115553258)

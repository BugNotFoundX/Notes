# web服务器的配置

- 以下是本文的部署环境：

    - 后端架构django==4.1.7

    - 服务器平台：阿里云

    - 服务器配置：
      - CPU&内存：2核(vCPU)2 GiB
          - 操作系统：Ubuntu 22.04LTS 64位
          - 实例规格：[ecs.t6-c1m1.large(性能约束实例)](https://help.aliyun.com/document_detail/25378.html#t6)
          - 实例规格族：ecs.t6
          - 当前使用带宽：1Mbps
          - 硬盘：40GB
- **这里省略连接服务器的步骤，推荐使用vscode的远程连接功能，方便改写文件**

## 一、环境配置

### 1、安装nginx

1. 使用命令行安装nginx

    (1) ubuntu命令如下

    ```shell
    sudo apt update
    sudo apt install nginx
    ```

    (2) CentOS命令如下

    ```shell
    sudo yum install -y epel-release
    sudo yum -y update
    sudo yum install -y nginx
    ```

    - CentOS可能遇到以下报错

        ```shell
        Error: Failed to download metadata for repo ‘appstream‘: Cannot prepare internal mirrorlist
        ```

        是由于CentOS 已经停止维护的问题。2020 年 12 月 8 号，CentOS 官方宣布了停止维护 CentOS Linux 的计划，并推出了 CentOS Stream 项目，CentOS Linux 8 作为 RHEL 8 的复刻版本，生命周期缩短，于 2021 年 12 月 31 日停止更新并停止维护（EOL），更多的信息可以查看 CentOS 官方公告。如果需要更新 CentOS，需要将镜像从 mirror.centos.org 更改为 vault.centos.org

        解决方法如下：

        - **首先**，进入到 yum 的 repos 目录

            ```shell
            cd /etc/yum.repos.d/
            ```

        - **其次**，修改 centos 文件内容

            ```shell
            sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
            sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
            ```

        - **然后**，生成缓存更新

            ```shell
            yum makecache
            ```

        - **最后**，运行 yum update 并重新安装nginx

    安装成功后，默认的网站目录为： `/usr/share/nginx/html`

    默认的配置文件为：`/etc/nginx/nginx.conf`

    自定义配置文件目录为: `/etc/nginx/conf.d/`

2. 查看是否安装成功

    ```shell
    nginx -v # 不同版本查看version的命令可能不同
    ```

    有如下字样按安装成功

    ```
    root@<your server name>:/# nginx -v
    nginx version: nginx/1.18.0 (Ubuntu)
    ```

3. 启动nginx服务

    ```shell
    service nginx start
    ```

    运行如下命令查看nginx的状态

    ```shell
    systemctl status nginx
    ```

    如下字样说明启动成功

    ```
    root@<your server name>:/# systemctl status nginx
    ● nginx.service - A high performance web server and a reverse proxy server
         Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
         Active: active (running) since Thu 2023-03-30 15:05:33 CST; 13min ago
           Docs: man:nginx(8)
        Process: 762 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
        Process: 986 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
       Main PID: 988 (nginx)
          Tasks: 3 (limit: 1929)
         Memory: 11.2M
            CPU: 33ms
         CGroup: /system.slice/nginx.service
                 ├─988 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
                 ├─989 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" 
                 └─990 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" 
    
    Mar 30 15:05:32 iZ0jlhcbu4ffvpc5zbtw4kZ systemd[1]: Starting A high performance web server and a reverse proxy server...
    Mar 30 15:05:33 iZ0jlhcbu4ffvpc5zbtw4kZ systemd[1]: Started A high performance web server and a reverse proxy server.
    ```

    nginx服务的控制命令

    ```shell
    /etc/init.d/nginx start   # 启动
    /etc/init.d/nginx stop    # 停止
    /etc/init.d/nginx reload  # 重新载入配置文件
    /etc/init.d/nginx restart # 重启
    
    service nginx [option] # 这样写也行
    
    # Usage: nginx {start|stop|restart|reload|force-reload|status|configtest|rotate|upgrade}
    ```

4. 配置阿里云安全组（其他厂商的服务器可能有不同，后续会有补充）

    ![](https://images.cnblogs.com/cnblogs_com/blogs/743211/galleries/2294100/o_230330072657_%E6%88%AA%E5%9B%BE%202023-03-30%2015-25-31.png)

    按照上图在安全组中添加规则

5. 在浏览器地址栏输入服务器公网ip，出现下图说明配置成功

    ![](https://images.cnblogs.com/cnblogs_com/blogs/743211/galleries/2294100/o_3c2f1c7d.png)

### 2、安装uwsgi

1. 通过 pip 命令：

    ```shell
    pip install uwsgi
    ```

2. 源代码安装：

    ```shell
    wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
    tar zxvf uwsgi-latest.tar.gz
    cd uwsgi-latest
    make
    ```

### 3、安装mysql

- 如果使用django默认的数据库，跳过这一步

1. TODO

## 二、项目配置

### 1、配置后端

#### (1)配置nginx

1. 不管用什么方法，将整个后端项目复制到/home目录下

    > 可以使用FileZilla软件

2. 进入/etc/nginx/conf.d目录

    ```ba
    cd /etc/nginx/conf.d
    ```

3. 新建一个 xxx.conf 文件，粘贴如下配置：

    ```nginx
    server {
    	listen 80;					   			# 在云服务器上配置安全组，开放80端口(监听http默认80端口)
      	server_name xxx.xxx.xxx.xxx;		   	# 云服务器的公网IP地址
        charset utf-8;
    
        # 后端项目媒体文件夹位置
        location /media {
            alias /home/backend/media;
        }
    
        # 后端项目静态文件夹位置
        location /static {
            alias /home/backend/static;
        }
    
        # 此项本质上是一个请求的转发，可配置多个，视后端url前缀种类而定
        location /api {							   # 与后端项目中url的前缀对应
            include /etc/nginx/uwsgi_params;
            uwsgi_pass 127.0.0.1:8000; 			   # 记住这里的端口，要与uwsgi的配置相匹配
        }
        
        # 以下是前端部署的部分，后面再说
        location / {
            root /home/frontend;
            index index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
        
        # 如果想要将请求转发给其他服务器可以添加如下配置
        location /proxy {
            proxy_pass http://xxx.xxx.xxx.xxx;
        }
    }
    
    ```

    **注意：location 后面是有空格的，必须要有！alias 后面也是有空格的；**

    tip：也可以采用软链接的方式，在你希望的目录创建xxx.conf文件，然后运行下面的命令（这个方法没有测试，不保证正确）

    ```shell
    sudo ln -s /home/backend/nginx.conf /etc/nginx/conf.d
    ```

4. 重启 nginx 服务

    ```bash
    sudo service nginx restart
    ```

#### (2)配置uwsgi

1. 在后端项目的根目录下，也就是有 manage.py 文件的目录下，新建一个 uwsgi.ini 文件和一个 run.log 文件

2. 编辑 uwsgi.ini 文件：

    ```ini
    [uwsgi]
    # 后端项目位置
    chdir = /home/backend
    # :application前面要换成自己的文件夹名称
    module = backend:application
    # 这里与上面conf文件中的端口要一样!
    socket = 127.0.0.1:8000
    # 项目中的wsgi-file路径
    wsgi-file = /home/backend/backend/wsgi.py
    master = true
    # 新建一个run.log文件记录信息
    daemonize = /home/backend/run.log
    # 运行uWSGI后,系统会在你填写的路径按照你设置的文件名创建一个.pid文件，并将uwsgi的进程号写入文件
    pidfile = /home/backend/uwsgi.pid
    disable-logging = true
    # 项目运行所需的虚拟环境，如果使用的是非虚拟环境则不需要这一行
    virtualenv = /root/.virtualenvs/venv
    
    ```

#### (3) 配置 MySql（若用的是自带数据库，则可跳过）

1. TODO

#### (4) 修改 settings.py 文件

1. 打开 settings.py 文件找到下面代码并修改：

    ```python
    DEBUG = False
    ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx']		# 改成['*']也行,即ALLOWED_HOSTS = ['*']
    ```

​	注意其中的 IP 地址要替换成你自己云服务器公网的 IP

2. 在文件末尾添加：

    ```python
    STATIC_ROOT= '/home/backend/static/'  # 你的静态文件存放的文件夹
    ```

#### (5) 进入虚拟环境，收集静态文件

- 激活你的项目的虚拟环境后输入指令收集静态文件:

    ```shell
    python3 manage.py collectstatic
    ```

#### (6) 运行后端

1. 在 `uwsgi.ini` 目录下运行

    ```bash
    uwsgi --ini uwsgi.ini
    ```

2. 如果出现

    ````
    [uWSGI] getting INI configuration from uwsgi.ini
    ````

​	则说明运行成功，可以通过公网ip访问你的服务

​	可以输入指令 `ps -aux | grep uwsgi` 来查看运行的进程

3. uWSGI控制相关终端指令

    ```shell
    sudo killall -9 uwsgi  #在Linux系统关闭已经在运行的uWSGI进程，用于重启服务
    uwsgi --ini uwsgi.ini  #启动
    uwsgi --stop uwsgi.pid  #关闭
    uwsgi --reload uwsgi.pid  #重启
    ```

    如果没有uwsgi.pid文件，可以手动创建，并将uwsgi的进程号（PID）写入文件，再次运行命令即可成功

### 2、配置前端

TODO

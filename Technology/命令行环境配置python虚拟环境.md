# 命令行环境配置python虚拟环境

- 本文操作系统：
    - ubuntu22.04.2TLS

## 一、虚拟环境的搭建

1. 先安装最基础的包virtualenv：

    ```shell
    pip install virtualenv
    ```

2. 安装安装virtualenvwrapper：

    ```shell
    pip install virtualenvwrapper
    ```

    如果出现报错

    ```
    ERROR: Could not find a version that satisfies the requirement pbr (from versions: none)
    ```

    则安装pbr后再安装virtualenvwrapper

    ```shell
    pip install pbr
    pip install virtualenvwrapper
    ```

3. 修改环境配置文件，在`.bashrc`或`.profile`文件中加入四行代码：

    ```shell
    export PATH
    export WORKON_HOME=$HOME/.virtualenvs
    export VIRTUALENVWRAPPER_PYTHON=/bin/python3
    source /usr/local/bin/virtualenvwrapper.sh
    ```

    source指向后缀为sh的脚本文件，在这里需要的是`virtualenvwrapper.sh`。同时要注意，不同系统中的路径可能不一样，find命令可以帮助我们找到需要的文件：

    ```shell
    find / -name virtualenvwrapper.sh
    ```

4. 编辑完后执行`source ~/.bashrc`重新加载配置文件即可正常使用命令。

## 二、虚拟环境的使用方法

### 1、mkvirtualenv

- 在WORKON_HOME目录中创建一个新的虚拟环境。

    ```shell
    mkvirtualenv [-a project_path] [-i package] [-r requirements_file] [virtualenv options] ENVNAME
    ```

- 参数说明：

    - -i：指定一个或多个包

    - -a：指定一个项目目录，用于将该虚拟环境绑定至一个现有项目

    - -r：指定一个requirements文件，一般是由`pip freeze`所生成的，里面包括了需要用到的所有包

    - virtualenv options：参数将直接传递给`virtualenv`，详情参阅[virtualenv官方文档](https://link.zhihu.com/?target=https%3A//virtualenv.pypa.io/en/stable/)

- 例如`mkvirtualenv env`会创建一个名为env的虚拟环境并激活，之后pip安装的所有包都仅仅存在于这个环境中。

### 2、mkproject

- 在PROJECTHOME目录中创建项目目录同时在WORKON_HOME目录下创建同名虚拟环境。

    ```shell
    mkproject [-f | --force] [-t template] [virtualenv_options] ENVNAME
    ```

- 例如`mkproject env`会创建一个名为`env`的项目目录，同时被创建的还有它的同名虚拟环境，当前虚拟环境会被切换到新的虚拟环境，当前工作目录也会改变为新的项目目录。

### 3、setvirtualenvproject

- 将现有的虚拟环境绑定到现有的项目。

    ```shell
    setvirtualenvproject [virtualenv_path project_path]
    ```

- 如果不指定参数，则默认为当前虚拟环境和当前目录作为参数，单个项目可以绑定多个虚拟环境，从而可以轻松地在不同的Python版本或依赖之间切换以进行测试。

### 4、rmvirtualenv

- 删除WORKON_HOME中的虚拟环境。

    ```shell
    rmvirtualenv ENVNAME
    ```

- 例如`mkvirtualenv env`将会删除虚拟环境`env`。

### 5、workon

- 列出所有虚拟环境或更改当前工作中的虚拟环境。

    ```
    workon [(-c|--cd)|(-n|--no-cd)] [environment_name|"."]
    ```

- 命令`workon`会列出所有可用的虚拟环境
- 命令`workon env`将会切换当前虚拟环境到虚拟环境`env`

### 6、deactivate

- 退出当前虚拟环境

    ```shell
    deactivate
    ```

### 7、其他

- 还有许多其他的命令，详情参阅[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/index.html)，以下仅列出一部分：

    - mktmpenv：创建一个临时的虚拟环境，停用后将会自动被清除

    - lsvirtualenv：列出所有可用环境

    - showvirtualenv：显示单个虚拟环境的详细信息

    - cdvirtualenv：将当前工作目录更改为虚拟环境的存放目录，可导航到子目录

    - cdproject：将当前工作目录更改为活动的虚拟环境所绑定的项目目录的目录

    - wipeenv：删除当前虚拟环境中所有已安装的第三方软件包
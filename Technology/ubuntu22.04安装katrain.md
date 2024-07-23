# ubuntu22.04安装katrain

```shell
# 用pip安装katrain包
pip install -U katrain

# 安装或配置相关依赖
sudo apt install xclip

cd /usr/lib/x86_64-linux-gnu/
sudo ln -s libzip.so.4 libzip.so.5

sudo apt-get install libssl1.1

# 启动，直接在命令行运行
katrain
# 第一次安装，稍作等待即可使用katrain，katrain会自动下载默认的katago模型并配置运行
```

一般来说默认的katago版本（按照上面的方法安装并启动时下载的版本）足以满足需求，如何切换katago内核没有进行更深入的研究


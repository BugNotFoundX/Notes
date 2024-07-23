# HUAWEI FreeLace Pro在ubuntu22.04.01上的蓝牙连接问题

- 可能是因为ubnutu缺少一个模块

  ```shell
  sudo apt install pulseaudio-module-bluetooth 
  pulseaudio -k
  pulseaudio --start # 一定要启动这个
  sudo pactl load-module module-bluetooth-discover # 这一行命令好像不成功也不影响
  ```

- 从命令行连接蓝牙的方法

  ```shell
  # 打开蓝牙的管理程序
  bluetoothctl
  
  # 扫描
  scan on
  # 信任
  trust XX:XX:XX:XX:XX:XX
  # 配对
  pair XX:XX:XX:XX:XX:XX
  # 连接
  connect XX:XX:XX:XX:XX:XX
  ```

- 注意耳机一定要处于配对模式（长按电源键4秒），否则ubuntu可能发现不了你的设备

- [参考链接](https://blog.csdn.net/weixin_48120620/article/details/126229978)
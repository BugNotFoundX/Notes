# Ubuntu桌面快捷方式模版

```ini
[Desktop Entry]
Name=			       		# 程序名字
Comment=		       		# 程序注释
Exec=			       		# 可执行文件的路径
Terminal=		       		# 是否打开终端[true, false]
Type=			       		# 应用类型，常用有Application等
Icon=						# 图标路径，支持png格式图片
Categories=		       		# 分类
Path=                       # 设置当前工作目录
X-GNOME-Autostart-enabled=	# 设置开机自启动[true, false]
StartupNotify= 				# 指示桌面环境是否应该在应用程序启动时显示通知[true, false]
NoDisplay =    				# 指示应用程序是否在应用程序菜单中显示[true, false]
```

-   Categories
    -   "AudioVideo"（音频和视频）：用于音频和视频播放器、编辑器等应用程序。
    -   "Development"（开发）：用于开发工具、集成开发环境（IDE）等应用程序。
    -   "Education"（教育）：用于教育和学习相关的应用程序。
    -   "Game"（游戏）：用于游戏应用程序。
    -   "Graphics"（图形）：用于图像编辑、绘图等图形应用程序。
    -   "Network"（网络）：用于网络工具和应用程序。
    -   "Office"（办公）：用于办公套件和办公工具应用程序。
    -   "Science"（科学）：用于科学研究和实验的应用程序。
    -   "System"（系统）：用于系统管理和系统工具的应用程序。
    -   "Utility"（实用工具）：用于各种实用工具应用程序。
-   Type
    -   "Application"（应用程序）：用于普通应用程序的类型。
    -   "Link"（链接）：用于指向其他应用程序、文档或URL的链接。
    -   "Directory"（目录）：用于表示该.desktop文件是一个目录，点击该目录时会打开文件管理器并显示该目录的内容。
    -   "Service"（服务）：用于表示该.desktop文件是一个系统服务的配置文件。
    -   "MimeType"（MIME类型）：用于表示该.desktop文件是与特定MIME类型相关联的应用程序配置文件，用于指定打开该类型文件的应用程序。

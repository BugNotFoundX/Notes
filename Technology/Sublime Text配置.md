# Sublime Text配置

## 一、通用配置

### 1.1 配置包管理工具

- 安装Package Control

### 1.2 配置中文显示

- 安装ChineseLocalizations包
- 重启sublime

### 1.3 配置copilot

- 安装LSP
- 安装LSP-copilot
- 重启Sublime
- 首次使用时，请按照以下步骤操作：
    1. 打开任何文件。
    2. 从命令面板执行`Copilot: Sign In`。
    3. 按照提示验证 LSP-copilot。
        1. `User Code`将自动复制到剪贴板。
        2. 将`User Code`粘贴到弹出的 GitHub 身份验证页面中。
        3. 返回 Sublime Text 并按`OK`对话框。
        4. 如果您看到“登录确定”对话框，则LSP-copilot 开始工作。

### 1.4 配置语法检查

- 安装SublimeLinter
- 按照需要配置不同的语法检查工具

### 1.5 终端

- 安装Terminus

- 添加配置快捷键如下

    ```json
    [
       { "keys": ["ctrl+shift+t"], "command": "toggle_terminus_panel" }
    ]
    ```

## 二、systemverilog/verilog环境配置

### 1.1 配置systemverilog/verilog语法高亮

- 安装SystemVerilog/Verilog

### 1.2 安装开发辅助工具

- 安装Verilog Gadget

- 常用功能如下

    | 名称                     | 功能             |
    | ------------------------ | ---------------- |
    | Instantiate Module       | 实例化模块       |
    | Generate Testbench       | 生成testbench    |
    | Insert Header            | 插入文件头部注释 |
    | Alignment (ctrl+shift+x) | 对齐             |

- 一个Header模板

    ```systemverilog
    // You can make a your own header file and set its path to settings.
    // (Preferences > Package Settings > Verilog Gadget > Settings - User)
    //
    //		"header": "Packages/Verilog Gadget/template/verilog_header.v"
    
    // -----------------------------------------------------------------------------
    // Copyright (c) 2014-{YEAR} All rights reserved
    // -----------------------------------------------------------------------------
    // Author : your name <your email>@email.com
    // File   : {FILE}
    // Create : {DATE} {TIME}
    // Revise : {RDATE} {RTIME}
    // Editor : sublime text{SUBLIME_VERSION}, tab size ({TABS})
    // Brief  : 
    // -----------------------------------------------------------------------------
    ```

### 1.3 配置语法解析

#### 1.3.1 verilator语法解析

- 安装SublimeLinter-contrib-verilator

- 修改SublimeLinter-contrib-verilator代码

    在Installed Packages目录下找到SublimeLinter-contrib-verilator.sublime-package中的liner.py文件的如下代码

    ```python
    extopt = self.settings.get('use_multiple_source', False)
    prjopt = self.settings.get('search_project_path', False)
    if extopt:
        if prjopt:
            prjfile = self.view.window().project_file_name()
            if isinstance(prjfile, str) and prjfile != "":
                prjdat = self.view.window().project_data()
                prjsrc = prjdat.get('sources')
                for path in prjsrc:
                    if sublime.platform() == 'windows':
                        if wslopt:
                            path = '/mnt/' + re.sub(':', '', path)
                        path = '-I' + re.sub(re.compile(r'\\'), '/', path)
    ```

    将以上代码修改为

    ```python
    extopt = self.settings.get('use_multiple_source', False)
    prjopt = self.settings.get('search_project_path', False)
    if extopt:
        if prjopt:
            prjfile = self.view.window().project_file_name()
            if isinstance(prjfile, str) and prjfile != "":
                prjdat = self.view.window().project_data()
                prjsrc = prjdat.get('sources')
                for path in prjsrc:
                    if sublime.platform() == 'windows':
                        if wslopt:
                            path = '/mnt/' + re.sub(':', '', path)
                        path = '-I' + re.sub(re.compile(r'\\'), '/', path)
                    else:
                        path = '-I' + path
                    cmd.append(path)
    ```

- 安装verilator

- 在SublimeLinter设置中配置verilator路径，下面是一个例子

    ```
    // SublimeLinter Settings - User
    {
    	"debug": true,
    
        "paths": {
            "linux": "/usr/local/bin/verilator_bin",
            "osx": [],
            "windows": []
        },
              
    }
    ```

- 更多配置如下

    ```
    {
        "no_column_highlights_line": true,
        "linters":
        {
            "verilator": {
                "lint_mode": "load_save",
                "styles" : [
                    {
                        "types": ["warning"],
                        "mark_style": "squiggly_underline",
                        "icon": "Packages/SublimeLinter/gutter-themes/Default/cog.png"
                    },
                    {
                        "types": ["error"],
                        "mark_style": "fill",
                        "icon": "Packages/SublimeLinter/gutter-themes/Default/cog.png"
                    }
                ],
                "args": [
                    "--error-limit",
                    "500",
                    "--default-language",
                    // "1800-2017", // systemverilog
                    "1364-2005", // verilog
                    "--bbox-sys",
                    "--bbox-unsup",
                    "-Wall",
                    "-Wno-WIDTH",
                    "-Wno-IGNINC",
                    "-Wno-IGNDEF",
                    "-Wno-STMTDLY",
                    "-Wno-UNDRIVEN",
                    "-Wno-PINCONNECTEMPTY",
                    "-Wno-INPUTPINEMPTY",
                    "-Wno-OUTPUTPINEMPTY"
                ],
                "filter_errors": [
                    "Unsupported:",
                    "\\[IGNDEF\\]",
                    // "expects 8192 bits" // not to use -Wno-WIDTH
                ],
    
                // to lint based on single file (ignoring external module definition)
                //   "use_multiple_source": false,
                //   "search_project_path": false,
    
                // to lint based on multiple files (searching external sources - the same directory or project path)
                //   "use_multiple_source": true,
                //   "search_project_path": true,
                //  example) example.sublime-project
                //       "sources": [ "D:\\project\\srcs", "D:\\project\\working" ]
    
                "use_multiple_source": false,
                "search_project_path": false,
    
                // windows subsystem for linux (wsl verilator_bin)
                "use_wsl": false,
    
                // additional option to filter file type
                "extension": [
                    ".v"
                ],
                // additional option for better highlighting near
                "message_near_map": [
                    ["Case values", "case"],
                    ["Suggest casez", "casex"]
                ]
            }
        }
    }
    ```


#### 1.3.2 vivado语法解析

-   安装SublimeLinter-contrib-xsim

-   在SublimeLinter设置中配置xvlog路径，下面是一个例子

    ```json
    // SublimeLinter Settings - User
    {
    	// "debug": true,
    	
    	// SublimeLinter Settings - User
    	"paths": {
           "linux": [
           	"/usr/local/Xilinx/Vivado/2019.2/bin"
           ],
           "osx": [],
           "windows": [],
        },
        "linters": {
            "xvlog": {
                "args": ["--relax"],
                "working_dir": "${TEMP:${project_path:${folder:$file_path}}}",
            },
            "xvlog_sv": {
                "args": ["--relax"],
                "working_dir": "${TEMP:${project_path:${folder:$file_path}}}",
            },
            "xvhdl": {
                "args": ["--relax"],
                "working_dir": "${TEMP:${project_path:${folder:$file_path}}}",
            }
        },
    }
    ```

-   可以在项目设置中引入所有的include文件所在目录

    ```json
    // .sublime-project
    {
        "folders":
        [
            {
                "path": "."
            }
        ],
        "settings": {
            // SublimeLinter-contrib-xsim
            "SublimeLinter.linters.xvlog.args":[
                "-i", "$project_path/PATH_TO_HEADER_0",
                "-i", "$project_path/PATH_TO_HEADER_1",
                "--relax"
            ],
            "SublimeLinter.linters.xvlog_sv.args":[
                "-i", "$project_path/PATH_TO_HEADER_0",
                "-i", "$project_path/PATH_TO_HEADER_1",
                "--relax"
            ],
            "SublimeLinter.linters.xvhdl.args":[
                "--relax"
            ]
        }
    }
    ```

    
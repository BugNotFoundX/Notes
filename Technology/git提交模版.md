# git提交模版

Git在提交时如果需要以指定的格式提交，可以给Git配置一个提交模板，配置好后，每次Git commit时都会将模板中的信息作为初始化信息自动填充到编辑器的提交信息中。

**以命令的方式配置：** `git config --global commit.template ~/.gitmessage.txt`

**在Git配置文件中配置：** 在Git配置文件的commit下添加一项template即可，Git配置文件通常在 `C:\Users\Administrator\.gitconfig`（Linux通常在`~/.gitconfig`）。其实以命令的方式执行后，它会自动在配置文件中生成这一项，Git commit的时候会去配置文件中读取这一项指定的文件中的内容，并把该内容作为初始化信息填写到提交信息中。

```text
[commit]
	template = C:/Users/Administrator/.gitmessage.txt
```

**注意：** 这里的模板文件路径是以斜杠的方式作为路径分隔符，即便是在Windows上，也是斜杠。而且文件本身只要是文本文件即可，不一定必须是txt文件，该文件也并没有规定必须放在什么位置，只需要将你实际的模板文件路径填写进去即可。

下面是[Angular提交信息规范](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)

```
# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component), e.g: route, component, utils, build...
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. Multiple lines separated by “-”, This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer: 
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#
# e.g:
# <type>(<scope>): <subject>
# <BLANK LINE>
# <body>
# <BLANK LINE>
# <footer>
# 
# types:
# feat:     A new feature 
# fix:      A bug fix 
# docs:     Documentation only changes 
# style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc) 
# refactor: A code change that neither fixes a bug nor adds a feature 
# perf:     A code change that improves performance 
# test:     Adding missing tests or correcting existing tests
# build:    Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm) 
# ci:       Changes to our CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs) 
# chore:    Other changes that don't modify src or test files 
# revert:   Reverts a previous commit
```


### 一、什么是WSL
Windows Subsystem for Linux [WSL 安装说明](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#for-anniversary-update-and-creators-update-install-using-lxrun)

### 二、WSL 启用
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

### 三、安装centos操作系统
#### 1. 下载centos提供的wsl包
[CentWSL](https://github.com/yuk7/CentWSL)

#### 2. 启动centos
- 将下载好的文件解压到本地，然后执行：CentOS7.exe （管理员方式运行），会在当前目录释放centos相关文件
- 运行完后可以点击CentOs7进入centos系统，或者将该目录添加至环境变量，支持执行CentOs7.exe
- WSL相关的信息参考注册表：计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss

### 三. pycharm与WSL结合
- 预置条件: pycharm 2018.3+专业版
- 参考链接：[Remote interpreter using WSL](https://www.jetbrains.com/help/pycharm/using-wsl-as-a-remote-interpreter.html)


### 问题汇总
- cmder vim方向键不可使用, 将配置中的startup改为：`%windir%\system32\wsl.exe ~  -cur_console:p5`
- 运行报错：.python-eggs is writable by group/others and vulnerable to attack when used with get_resource_filename。 执行以下命令：`chmod g-wx,o-wx ~/.python-eggs`
- 缺失文件：动态链接库、系统初始化相关配置文件，将37服务器相关文件拷贝

# 怎么在M1 Mac上使用libTAS

## 安装带有rosetta的Ubuntu系统
在Parallels可以直接下载，这里省略，如果不是自己查找资料解决

## 在Mac上安装Rosetta
首先你需要在你的系统上安装rosetta:

```
softwareupdate --install-rosetta --agree-to-license
```

然后你现在就应该有rosetta的文件了:

```
$ tree /Library/Apple/usr/libexec/oah
/Library/Apple/usr/libexec/oah
├── RosettaLinux
│   ├── rosetta <- 这个
│   └── rosettad
├── debugserver -> /usr/libexec/rosetta/debugserver
├── libRosettaRuntime
├── runtime -> /usr/libexec/rosetta/runtime
└── translate_tool -> /usr/libexec/rosetta/translate_tool
```

__注意__：如果您没有获得`RosettaLinux/rosetta`二进制文件，请先尝试遵循[UTM Rosetta](https://docs.getutm.app/advanced/rosetta/)指南，该指南应该会为您安装Rosetta

## 在Ubuntu上安装Rosetta
把`RosettaLinux/rosetta`复制到你的虚拟机里.在这个范例里，我们把该文件复制到了`/bin/rosetta`里.实际上，在该程序任何路径中都应该有效

```
$ /bin/rosetta
rosetta error: Rosetta is only intended to run on Apple Silicon with a macOS host using Virtualization.framework with Rosetta mode enabled
Trace/breakpoint trap (core dumped)
```

这个错误是意料之中的

### 使用`mount-rosetta.py`

要修复错误，您可以使用mount-rosetta.py创建一个FUSE挂载，该挂载实现了Rosetta握手所需的ioctl:

```
sudo apt install libfuse2
sudo python3 -m pip install fusepy
```

__注意__：要在没有root的情况下挂载，请编辑`/etc/fuse.conf`并取消注释`user_allow_other`.除非您正在开发脚本，否则这应该没有必要.

现在，你可以运行:

```sudo python mount-rosetta.py /bin/rosetta```

这将跟踪`/bin/rosetta`二进制文件，并处理必要的ioctl调用.为了确认程序可运行，您可以运行：

```
$ /bin/rosetta
Usage: rosetta <x86_64 ELF to run>

Optional environment variables:
ROSETTA_DEBUGSERVER_PORT    wait for a debugger connection on given port

version: Rosetta-289.7
```

现在你就成功了！

__注意__: 运行`/bin/rosetta`时，应在后台挂载python运行的`mount-rosetta.py`

## 安装libTAS
阅读[这里](https://github.com/clementgallet/libTAS)安装libTAS

在安装libTAS后，直接运行会触发`Exec format error`错误，需要运行`rosetta /bin/libTAS`以运行libTAS

## 运行游戏
拥有`Gamemaker studio 2`的，可以自行打包一个Ubuntu的可执行文件

__注意__: Arch架构不能打包出`.AppImage`文件.在填写文件名时，应把后缀改成`.zip`

打包出来的文件里面会有`assets`和一个可执行文件，运行可执行文件后，会出现`libcrypto.so.1.1`不存在,或`libssl.so.1.1`不存在的错误，只需在该库中下载下来然后拷贝或移动到`/lib`

之后可以运行`rosetta xxx`启动游戏，`xxx`为可执行文件名称

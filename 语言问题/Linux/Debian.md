# 换源

## debian、ubuntu

```shell
sudo vim /etc/apt/sources.list
```

这里是可以用的

```Shell
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse


deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
```

## centos

```Shell
# 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

```Shell
# 创建并且写入
sudo vim /etc/yum.repos.d/CentOS-Base.repo 
```

```Shell
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#
 
[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```



# Nvidia 驱动的安装

主要的安装 [参考这篇文章在线安装](https://blog.csdn.net/ROSEBUD7_K/article/details/126172572)



亮度调节：

之前装 510 驱动的时候，通过 grub 设置，可以调节亮度

后面装 470 驱动的时候，发现不能调节了，可能是驱动的问题

执行下面这条命令可以修改亮度，不过 DP-4 需要手动改，可以在 NVIDIA X Server Settings 查看

可以加在`~/.bashrc`中

```Shell
xrandr --output DP-4 --brightness 0.6
xrandr --output HDMI-0 --brightness 0.6
```



# 设置语言

前置操作记不清了，反正可以在系统设置的语言里面进行切换

在这个目录下`/etc/default/locale`，不过之前需要下载英文，中文包，需要啥下啥

如果想在终端输出英文，需要在配置文件中修改

```Shell
vim ~/.bashrc
```

```Shell
# 把终端输出设置成英文
export LANGUAGE=en_US 
export LANG=en_US.UTF-8 
```

# 安装以及卸载

安装方式有两种，一种是通过下载 deb 包，然后进行安装，另外一种是直接 apt 安装

- dpkg安装

```Shell
# 安装 deb包，会安装在 /opt 路径下
sudo dpkg -i XXXXXX.deb
# 卸载，首先查找要卸载的全部名称
sudo dpkg -l | grep XXX
# 然后根据名称，直接全部卸载
sudo dpkg --purge XXXXX
```

- apt 安装

```Shell
# 安装，会安装在
sudo apt install XXXX
# 卸载软件及其配置 
apt-get --purge remove  XXXX
```

```Shell
# 下载的软件的存放位置
/var/cache/apt/archives
# 安装后软件的默认位置
/usr/share
# 可执行文件位置
/usr/bin
# 配置文件位置
/etc
# lib文件位置
/usr/lib
```

# 创建快捷方式

在 `/usr/share/applications`下面，创建一个 **XXX.desktop**文件，[linux图形化应用程序快捷方式制作方法](https://blog.csdn.net/shixin_0125/article/details/105385465)

```Shell
[Desktop Entry]
Encoding=UTF-8
Name=应用程序名
Exec=执行应用程序可执行文件的命令
Icon=图标路径
Terminal=false
StartupNotify=true
Type=Application
Categories=Application;
```



# 常用的软件安装

## 火狐

首先安装火狐，然后配置好下载路径



## 搜狗

输入法的安装先到官网找



## Qv2ray

翻墙软件也是必备的啦

可能会出现问题，进入到 `/etc/profile`里面，加入语句

```Shell
sudo vim /etc/profile

# add Qv2ray
export DISPLAY=:0.0
```



## Chrome

直接官网下载包安装

[调整成暗黑背景](https://blog.csdn.net/iamlujingtao/article/details/108887919)



## Typora

笔记软件



## PicGo

这里提供两种方法：第一种是去官网下载 appImage，直接运行即可，第二种是不下载，直接配置，但是**没成功**

- **APPImage**

[PicGo官网](https://github.com/Molunerfinn/PicGo/releases) 去下载发行版，选择 appimage，这样下载后直接双击就能运行，配置PicGo，下面是一些配置以及相应的 token

```Shell
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
# 加速
https://cdn.jsdelivr.net/gh/vaesong/Images/
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20220925093003.png)

按照之前填的发现不能运行，报错

![](https://raw.githubusercontent.com/vaesong/Images/master/20220925092803.png)

这里是需要**下载 xclip**

```Shell
sudo apt install xclip
```

然后就能愉快的上传图片了，**如果还是不行的话，看看是不是上传了同名文件**

碰到问题不要瞎搞，能看日志还是要看日志

- nodejs，typora自动上传

这个方法我没成功，成功的话，其实更方便，直接复制就会自动上传了。如果要改的话，github 的插件是 github-plus

[ubuntu(Linux)配置Typaro+PicGo+gitee实现图片上传云端](https://lukeyalvin.blog.csdn.net/article/details/117414129?spm=1001.2101.3001.6650.14&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-14-117414129-blog-106796119.pc_relevant_multi_platform_featuressortv2dupreplace&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-14-117414129-blog-106796119.pc_relevant_multi_platform_featuressortv2dupreplace&utm_relevant_index=15)



## 微信、QQ

参考这两篇文章[CSDN博客](https://blog.csdn.net/weixin_46584887/article/details/121961101)、[deepin-wine的github](https://github.com/zq1997/deepin-wine#%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B)

按照教程安装完成后，微信整个界面周围出现黑框

后面自己没了......



## WPS

官网下载



## Vscode

安装Git

官网下载，其中对于之前的配置，需要进行同步，参考下面这篇博客，可以同步已经安装的插件，但是也只能同步插件了

我的 ssh 配置等，都无法同步

[VsCode同步配置到另一台设备](https://blog.csdn.net/qq_39742704/article/details/122908148?spm=1001.2014.3001.5501)

下面是我的 gist_id 和相应的 token，因为 token 只能看一次，所以需要进行保存

```Shell
Token：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

Gist ID：83cc7225b163087b02a67f919136f56c
```

但是需要关闭 sync 的自动同步，首先点击用户设置，`setting.json`

在里面搜索 `autoUpload` 和 `autoDownload` 设置为 false 就行

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20220926162736.png)



**设置 vscode 的字体**，是 fira code。[字体](https://www.cnblogs.com/langkyeSir/p/14018116.html)

```Shell
# 执行这个命令安装，然后后续使用脚本
sudo apt install fonts-firacode
```

这里就直接装好了，不需要下面的脚本

到 Vscode 设置里面，搜索 `Front Family` 进行修改，改为 `Fira Code`



## 腾讯会议

官网下载即可

在用 dpkg 进行安装的时候，出现错误

```Shell
dpkg: error processing package ***(--install): dependency problems - leaving unconfigured
```

依赖错误, 安装相应的依赖即可. 或者自动安装缺少依赖:

```Shell
sudo apt install -f
```



## 网易云音乐

官网下载，但是安装之后出现问题，[参考这里](https://zhuanlan.zhihu.com/p/460572309)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20220925142242.png)

```Shell
sudo rm /opt/netease/netease-cloud-music/libs/libselinux.so.1
sudo cp /lib/x86_64-linux-gnu/libselinux.so.1 /opt/netease/netease-cloud-music/libs
```

但是还是不能解决，有些歌曲无法播放的问题，原因好像是**不能播放 flac 类型**的音乐

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20220925143634.png)

[Linux 网易云音乐问题汇总](https://blog.csdn.net/u011715038/article/details/109699708)

两个方法都试了，还是不行！！！



## 截图软件

flameshot，参考下面这篇博客

[Linux上好用的截图工具——flameshot](https://blog.csdn.net/qq_36393978/article/details/102402698)

我在这里安装的时候，出现下面这种错误：

```Shell
Error! Bad return status for module build on kernel: 5.10.0-18-amd64 (x86_64)
Consult /var/lib/dkms/bcmwl/6.30.223.271+bdcom/build/make.log for more information.
dpkg: error processing package bcmwl-kernel-source (--configure):
 installed bcmwl-kernel-source package post-installation script subprocess returned error exit status 10
Errors were encountered while processing:
 bcmwl-kernel-source
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

经过搜索后得到答案，运行下面的几条命令即可，然后就可以继续按照博客搭建

```Shell
sudo rm -f /var/lib/dpkg/info/initramfs-tools.post*
sudo rm -f /var/lib/dpkg/info/initramfs-tools.pre*
 
sudo rm -f /var/lib/dpkg/info/bcmwl-kernel-source.post*
sudo rm -f /var/lib/dpkg/info/bcmwl-kernel-source.pre*
 
sudo dpkg --configure -a
```

设置全局快捷键像下面这张图



## 视频播放器

```Shell
sudo apt install mpv
```



## 录制GIF的软件

```Shell
sudo apt install peek
```



## 向日葵

远程控制软件，官网下载即可



## Zotero

文献管理软件，官网下载即可，不过是压缩包，还需要手动解压等操作

[在Linux 系统下安装zotero](https://blog.csdn.net/aiboom/article/details/123245533)

**存储路径**在**高级**里面的**文件和文件夹**下的**数据存储位置**



## OBS（录屏软件）

```Shell
# 打开终端添加PPA:
sudo add-apt-repository ppa:obsproject/obs-studio
# 软件更新
sudo apt-get update
# 安装OBS-studio
sudo apt-get install obs-studio
```

在安装软件包的时候，发现执行add-apt-repository命令会出现错误，无法添加软件源

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221006164133.png)

修改`/usr/lib/python3/dist-packages/softwareproperties/SoftwareProperties.py`文件，第777行的`isAlive`改成`is_alive`即可



## Xdm

似乎只能下载视频...

[linux（ubuntu）之下的下载神器XDM(xtreme download manage)](https://www.jianshu.com/p/d3d35fd7bb8d)



## Fiddler(mono环境下的太老了，已经放弃)

[在linux下安装工具fiddler](https://blog.csdn.net/weixin_40894428/article/details/80696771)

然后创建一个快捷方式



## Wireshark

[Linux抓包（wireshark+tcpdump）](https://blog.csdn.net/weixin_53946852/article/details/125656088)

```Shell
# 安装
sudo apt -y install wireshark* -y
```

过程中出现的蓝色界面选 yes，然后可能会提示说权限不够

```Shell
# 添加用户组
sudo groupadd wireshark
# 将dumpcap更改为wireshark用户组
sudo chgrp wireshark /usr/bin/dumpcap
# 让wireshark用户组有root权限使用dumpcap
sudo chmod 4755 /usr/bin/dumpcap
# 将自己加入wireshark用户组
sudo gpasswd -a vaesong wireshark
```





# 美化教程

这里出现了点问题，之前设置了很多东西，但是根本没有反应，然后突然所有的设置一起出现（能够使用）

是下面这个原因

- 到系统设置里设置一下 **混成器**，因为驱动的问题，OpenGL 的渲染会崩溃，改一下之后，重启就行



## 设置全局代理

在后续使用商店下载的过程中，下载不了主题，连接不到服务器

终端等一些使用，可能不会走全局代理，需要下载辅助工具

```Shell
sudo apt-get install proxychains
```

进入

```Shell
sudo vim  /etc/proxychains.conf
```

注意这里需要把 proxy_dnf 屏蔽掉，这样不让其走自身的域名解析服务

然后在最后的 sock 改为自己的代理地址，注意是 sock 端口，而不是 http 端口

```Shell
socks5 127.0.0.1 1089
```

后续终端使用的时候，只要在前面加上 proxychains 就能走全局代理了

```shell
proxychains systemsettings5
```

当系统设置打不开的时候，使用命令强制关闭

```Shell
killall systemsettings5
```

**全局代理**

[参考这篇文章](https://zhuanlan.zhihu.com/p/369356836)

在 `/etc/profile`中，添加命令，这里的 IP 与端口，和使用的翻墙软件保持一样

```Shell
export https_proxy=http://127.0.0.1:8889 
export http_proxy=http://127.0.0.1:8889 
export all_proxy=socks5://127.0.0.1:1089
```



## 美化主题

- 全局主题	WhiteSur-dark
- Plasma 样式     WhiteSur-dark
- 窗口装饰      WhiteSur-dark   按钮大小可设置



## alt+tab 切换

在窗口管理的**任务切换器**

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20220925104716.png)

## latte-dock

下载 latte-dock，可以在底部设置类似 Mac 的停靠栏的风格

然后点击布局、配置、下载，选的是 mcOS BS Layout for Latte Dock

## 小插件

查看网速的，调整颜色	netspeed widget

上方添加 	图标任务管理器

panon 音乐可视化，会出现问题

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20220925095531.png)

我的建议是去[官网](https://github.com/rbn42/panon)，之前跟着别人 arch 的教程搞半天，结果装的东西不是一样的！！！！

```Shell
sudo apt-get install qml-module-qt-websockets python3-docopt python3-numpy python3-pyaudio python3-cffi python3-websockets
```

设置：我采用的**视觉特效**是第一个，设置是 5、2、0.003

**音频数据源是 PluseAudio**，下面选择自己的音源

## 动态壁纸

下面是安装解析视频的依赖包

```Shell
sudo apt-get install libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools
```

# 开发环境

## Anaconda

下载脚本，在 zp_3c 服务器上的 home 下面

然后运行，anaconda创建成功

```Shell
# 创建环境
conda create --name script python=3.6
```

然后换源

```Shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

# 查看是否添加成功
conda config --show
```

## Linux 下 C++ 

其实最主要的就是能够进行调试，两个文件，一个是 `lauuch.json`，另外一个是 `tasks.json`

-  `tasks.json`按 F5，然后再点击C++（GDB/LLDB）-> C/C++: **g++ 生成活动文件**，下面是配置

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++ 生成活动文件",
            "command": "/usr/bin/g++",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": "build",
            "detail": "调试器生成的任务。"
        },
        {
            "type": "cppbuild",
            "label": "C/C++: cpp 生成活动文件",
            "command": "/usr/bin/cpp",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

-  `lauuch.json` 按 F5，然后再点击 C++（GDB/LLDB）-> C/C++: **cpp 生成活动文件**，下面是配置

```Json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++ 生成活动文件",
            "miDebuggerPath": "/usr/bin/gdb",
            "miDebuggerArgs": "-q -ex quit; wait() { fg >/dev/null; }; /bin/gdb -q --interpreter=mi",
        }
    ]
}
```

这里的 launch.json 设置之后，可能进行调试的时候，看不到容器内部的值：

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221006114942.png)

经过查找，发现是`launch.json`需要进行设置，[参考这里](https://github.com/Microsoft/vscode-cpptools/issues/1414)

需要在修改`setupCommands`这里面的值

```C++
"setupCommands": [
    {
        "description": "Test",
        // 这里的位置需要修改,似乎又不需要修改...,总之看一下
        "text": "python import sys;sys.path.insert(0, '/usr/share/gcc/python');from libstdcxx.v6.printers import register_libstdcxx_printers;register_libstdcxx_printers(None)",
        "ignoreFailures": false
    },
    {
        "description": "Enable pretty-printing for gdb",
        "text": "-enable-pretty-printing",
        "ignoreFailures": true
    }
]
```

但是注意，这里的 Test 里面插入的 Python，需要修改路径地址，似乎又不需要修改...,总之看一下`/usr/share/gcc`路径下是否有`python`，如果没有的话

```Shell
# 首先到路径下
cd /usr/share/gcc
# 首先安装 svn
sudo apt install subversion -y
# 然后搞一下 Python
svn co svn://gcc.gnu.org/svn/gcc/trunk/libstdc++-v3/python
```

然后出现说此 gdb 副本不支持 python 脚本

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221006141552.png)

这里查了一些问题，似乎是 gdb 的版本有问题，大部分的解决是下载包之后，在构建的时候，with Python

这里有点麻烦，所以又查到了另外一种方法，执行下面这条命令即可。[这里是博客链接](https://blog.csdn.net/rznice/article/details/108743791)

```Shell
sudo apt install libc6-dbg gdb valgrind
```

然后就能愉快的使用了

# 控制台设置颜色

家目录下的配置文件

~/.bashrc

```bash
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```



# Linux编译运行C++程序

```C++
g++ -o test test.c
test.exe
```



# 添加用户 useradd

常见用法

```Shell
sudo useradd 3c -m -d /data1/3c -s /bin/bash
sudo ln -s /data1/3c /home/3c
# 到 /etc/passwd 文件中修改家目录
sudo vim /etc/passwd

#永久性删除用户账号
userdel <用户名称>
```



```Shell
用法：useradd [选项] 登录
      useradd -D
      useradd -D [选项]

选项：
  -b, --base-dir BASE_DIR       新账户的主目录的基目录
  -c, --comment COMMENT         新账户的 GECOS 字段
  -d, --home-dir HOME_DIR       新账户的主目录 （要与 -m 配合使用）
  -D, --defaults                显示或更改默认的 useradd 配置
  -e, --expiredate EXPIRE_DATE  新账户的过期日期
  -f, --inactive INACTIVE       新账户的密码不活动期
  -g, --gid GROUP               新账户主组的名称或 ID
  -G, --groups GROUPS   新账户的附加组列表
  -h, --help                    显示此帮助信息并推出
  -k, --skel SKEL_DIR   使用此目录作为骨架目录
  -K, --key KEY=VALUE           不使用 /etc/login.defs 中的默认值
  -l, --no-log-init     不要将此用户添加到最近登录和登录失败数据库
  -m, --create-home     创建用户的主目录
  -M, --no-create-home          不创建用户的主目录
  -N, --no-user-group   不创建同名的组
  -o, --non-unique              允许使用重复的 UID 创建用户
  -p, --password PASSWORD               加密后的新账户密码
  -r, --system                  创建一个系统账户
  -R, --root CHROOT_DIR         chroot 到的目录
  -P, --prefix PREFIX_DIR       prefix directory where are located the /etc/* files
  -s, --shell SHELL             新账户的登录 shell
  -u, --uid UID                 新账户的用户 ID
  -U, --user-group              创建与用户同名的组
  -Z, --selinux-user SEUSER             为 SELinux 用户映射使用指定 SEUSER
```





# 用户组

```shell
# 已有用户添加到用户组， -a 表示 append， -G 表示用户组的名称
usermod -G <group名称> -a <用户名>

# 查看用户组
grep <group名称> /etc/group

# 将用户从该用户的附属组中删除
gpasswd -d <用户名称> <用户组名称>
```







# Linux 不能图形化问题

- 查看Xshell连接的时候 是否有问题 [链接](https://blog.csdn.net/wugenqiang/article/details/86554753)
- 然后检查Xmanager 是否安装 [链接](https://www.pianshen.com/article/80871587504/)
- 然后是 Xshell 和 Xmanager的配合使用 [链接1](https://blog.csdn.net/qq_25406563/article/details/82082273) [链接2](https://blog.csdn.net/qq_42283110/article/details/84889466)
- 打开 Xmanager - Passive
- xhost +无法打开的问题 [链接](https://blog.51cto.com/77jiayuan/2160835)

> 执行以下命令
> `vncserver`
> `export DISPLAY=localhost:1`
> `xhost +`

- make qemu 内存分配的问题 [链接](https://blog.csdn.net/szuhuanggang/article/details/88770938)

> 执行以下命令
> `sudo sysctl vm.overcommit_memory=1`



# 服务器不能Vscode不能显示

使用 `vscode` 进行远程登录的情况下，对于一些需要图像化显示的问题，总是会报 qt 的错误

```Shell
qt.qpa.xcb: could not connect to display 
qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland, wayland-xcomposite-egl, wayland-xcomposite-glx, webgl, xcb.
```

这种情况下需要使用 `xvfb-run` 进行启动，但是使用这个命令的话，就没有办法进行调试了，因为没找到调试的参数可以加 `xvfb-run`

后来经过查找，可以在终端 `ssh` 的时候，加上 `-X` 参数，或者在 `vscode` 的 `remote ssh` 时候，向下面一样配置，加上下面三行 yes ，这样的话，就不需要加上 `xvfb-run` 也能运行了，也就可以调试了

还需要执行下面的命令，但是，我怎么知道是 11.0 呢，而不是 0.0？ `export -p` 可以查看

破案了，这里其实是把服务器的显示给转发到了我的 `Linux` 上。[DISPLAY的解释](https://blog.csdn.net/landdin2013/article/details/48264831)

```shell
export DISPLAY=localhost:11.0
```

使用下面的命令去查找，但是好像要设置了，才能找到...可是一开始怎么知道如何设置呢（这个只有在正确的连接之后，才能看到）

```Shell
# 应该是过滤 display
xdpyinfo | grep display
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221027144923.png)

![image-20221027145309572](/home/vaesong/.config/Typora/typora-user-images/image-20221027145309572.png)

`vscode` 安装扩展 `Remote X11 (SSH)` ， 好像不用 SSH 的版本

```shell
Host liuchang
    HostName 120.76.100.14
    Port 6001
    ForwardX11 yes
    ForwardX11Trusted yes
    ForwardAgent yes
    User liuchang
```

补充： xhost 的安装

```Shell
sudo apt-get install x11-xserver-utils
```



# 服务器的 tensorboard 可视化

```Shell
ssh -L 本地端口:127.0.0.1:TensorBoard端口 用户名@服务器的IP地址 -p 服务器登录端口
```



# Linux下查看环境变量

```shell
echo $PATH
```



# 服务器卡顿

**CPU是不是很高**： 首先看一下 CPU 是不是占用率很高

```Shell
top
# 或者 
htop
```



**物理内存是不是用完了**： 发现不是很高的情况下，查看系统内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存

```Shell
free -h
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230410200556.png)

- **free** 列显示还有多少物理内存和交换空间可用使用
- **buff/cache** 列显示被 buffer 和 cache 使用的物理内存大小
- **available** 列显示还可以被应用程序使用的物理内存大小
- available <= free + buff/cache 因为有一部分page或cache是不能回收的，因为 **可使用内存 = 剩余内存 + 缓存内存**

可以看到 还可以申请的物理内存 = available 比较大，但是 `free` 很少，说明剩余的物理内存不多了，但是缓存了很多



**然后看 IO 是不是很高**： 查看磁盘IO使用情况

```Shell
# 后面那个1表示一秒刷新一次
iostat -x 1
# 没有的话安装
sudo apt install sysstat
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230410201312.png)

当发现最右侧%util很高时，表示IO就很高了。可以看到，这里磁盘的 IO 很高很高，由上可知：目前有多块物理磁盘，**sdb磁盘的io压力较大**

**使用`iotop` 命令查看当前的 IO 情况**，没有的话先安装

```shell
sudo apt install iotop
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230411104354.png)

可以看到，磁盘读数据的压力很大，尤其是这几个 python 程序

**然后检查sda磁盘中哪个应用程序占用的io比较高**

```shell
sudo pidstat -d  1
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230411105411.png)

可以看到，几个 python 程序一直读磁盘，这里选 2424884 查看。

**分析应用程序中哪一个线程占用的io比较高**

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230411105749.png)

由上可知：2424884 这个线程占用的io比较高

**分析这个线程在干什么？**

```shell
sudo perf trace -t 2424884 -o /tmp/tmp_aa.pstrace
cat /tmp/tmp_aa.pstrace
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230411110020.png)

目前这个线程在读文件，fd为文件句柄，文件句柄号为 82

**查看这个文件句柄是什么**

lsof 查看进程打开的文件

```shell
sudo lsof -p 2424884 | grep 82
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230411110340.png)

这个线程在疯狂从磁盘读文件， 是 `dataloader` 加载数据的文件，查看其大小

```shell
du -h XXXXX
```

![image-20230411110637354](/home/vaesong/.config/Typora/typora-user-images/image-20230411110637354.png)

也不是很大，初步估计是 `pin_memory` 没有设为 `True` ，导致没有把数据加载到物理内存中

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230411110807.png)





# 使用iftop查看带宽占用情况

如果服务器带宽跑满了，查看跟哪个ip通信占用带宽比较多，还可以用来监控网卡的实时流量（可以指定网段）、反向解析IP、显示端口信息等

```Shell
yum install -y iftop
```

安装好后在服务器执行iftop -i eth1就可以查看服务器公网网卡带宽使用情况(如果只执行iftop默认检测第一块网卡使用情况，这样查的会是内网网卡eth0。

```Shell
iftop -i eth0 -P
```







# vscode单击右键容易误触(自动点击)

[参考这篇博客](https://blog.csdn.net/qq_42330920/article/details/125778044)

`easystroke`

# Git连接github进行提交

参考这两篇文章[Git上传文件代码到GitHub（超详细）](https://blog.csdn.net/weixin_43806049/article/details/124963415)、[手把手教你用git上传项目到GitHub](https://zhuanlan.zhihu.com/p/193140870)

这篇博客超级详细的解释了原理 [Git 原理详解及实用指南](https://blog.csdn.net/Aria_Miazzy/article/details/107782890)

首先安装 git，然后配置用户名和邮箱

```Shell
git config --global user.name "vaesong"
git config --global user.email “18365519973@163.com”
```

还需要绑定一个 `.ssh` 下面的公钥，如果没有，需要手动创建，然后**把公钥复制到`github`上面的 ssh 设置里面**

```Shell
ssh-keygen -t rsa -C "18365519973@163.com"
ssh-keygen -t rsa
```



其中最主要的操作就是：

```shell
git init //把这个目录变成Git可以管理的仓库
git add README.md //文件添加到仓库
git add . //不但可以跟单一文件，还可以跟通配符，更可以跟目录。一个点就把当前目录下所有未追踪的文件全部add了 
git commit -m "first commit" //把文件提交到仓库
git remote add origin git@github.com:vaesong/Note.git //关联远程仓库,只需要第一次做
git push -u origin main //把本地库的所有内容推送到远程库上
```

如果出现`error: remote origin already exists.`这种情况，需要先删除关联的 origin 的远程仓库

```Shell
git remote rm origin
```

实际上在进行 git 上传的时候，github并没有完全把文件显示，是因为有些文件夹是空的，必须有东西时，才会上传成功，单个文件夹是无法上传成功的！！！

可能在 `pull` 的时候还会碰到冲突的情况

- 将本地修改放入**缓存区**(成功后本地**工作区间**的代码跟**本地仓库**代码会同步)

```shell
git stash 
```

- 从**远程仓库**获取最新代码

```Shell
git pull
```

- 然后， 取出本地修改的代码

```Shell
git stash pop
```

之后，再根据提示的冲突文件，进行手工修改



# 两台服务器之间传输文件夹

[参考这里](https://blog.csdn.net/qq_42224274/article/details/81735315?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-81735315-blog-125486150.pc_relevant_multi_platform_whitelistv5&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-81735315-blog-125486150.pc_relevant_multi_platform_whitelistv5&utm_relevant_index=1)

注意这里是大 P

```shell
scp -P [目标服务器的端口] [要传输的文件路径] [目标用户名]@[目标IP]:[目标文件路径]
scp -P 13122 Matterport3D.tar lhui@166.111.138.88:/home/lhui
如果是文件夹的话，前面加上 -r 
# 这个是从服务器下载文件到本地，这里指定的端口是服务器的端口
scp -P 13122 zp_3c@166.111.138.88:~/Anaconda3-2020.02-Linux-x86_64.sh /home/vaesong
```

本地与服务器之间传输文件，[点击这里](https://www.cnblogs.com/doubleyue/p/14688181.html)

使用frp搭建的内网穿透不能用 scp 命令了，使用 sftp

```Shell
# sftp username@ip 
sftp username@ip
# 将文件上传到服务器上
put [本地文件的地址] [服务器上文件存储的位置]
# 将服务器上的文件下载到本地
get [服务器上文件存储的位置] [本地要存储的位置]
```

# 挂载U盘

首先查看所有的磁盘信息

```Shell
sudo fdisk -l
```

然后找到 U 盘，通常是 `/dev/sd*`

```Shell
mkdir /mount
sudo mount /dev/sdb1 /mount
```

最后取消挂载

```Shell
sudo umount /mount
```

期间可能会出现不能识别 U 盘格式的问题，需要安装两个东西

```Shell
sudo apt-get install exfat-fuse exfat-utils
```

再次尝试，如果还是不行，就执行下面的命令

```Shell
sudo mount.exfat-fuse /dev/sdb1 /mount
```



# 删除运行某个文件的所有进程

例如，这里的 `grep -v grep` 的作用是过滤掉 `grep` 这条命令。`-v` 表示不选择

```Shell
ps -aef | grep 'dataset_generator.py' | grep -v grep | awk '{print $2}'
kill -9 `ps -aef | grep 'dataset_generator.py' | grep -v grep | awk '{print $2}'`
```



# 双系统安装Debian

## 分区大小的设置

```Linux
/
linux系统根目录
目录结构如下：
linux目录结构：

/bin

存放二进制可执行文件(ls,cat,mkdir等)，系统常规命令一般都在这里。

/sbin

存放二进制可执行文件，只有root才能访问。这里存放的是系统管理员使用的系统级别的管理命令和程序。如ifconfig等。

/dev

用于存放设备文件。

/home

存放所有用户文件的根目录，是用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示

/root

超级用户家目录。

/lib

存放跟文件系统中的程序运行所需要的共享库及内核模块。共享库又叫动态链接共享库，作用类似windows里的.dll文件，存放了根文件系统程序运行所需的共享文件。系统函数库。

/lib
系统64位函数库。

/proc

虚拟文件系统目录，是系统内存的映射。可直接访问这个目录来获取系统信息。

/run
当前运行的硬件和系统模块。

/srv
系统数据（常量）

/var

用于存放运行时需要改变数据的文件，也是某些大文件的溢出区，比方说各种服务的日志文件(系统启动日志等。)等。系统数据（变量）。

/tmp

用于存放各种临时文件，是公用的临时文件存储点。


/mnt

系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统。临时设备挂载点。

/media
cdrom（光驱）临时挂载点。

/etc

存放系统管理和配置文件。

/opt

第三方软件安装位置。额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把tomcat等都安装到这里。

/boot

存放用于系统引导时使用的各种文件。

/sys
关于内核设定目录。

/usr
（unix system resourse）仅次于/目录，多数系统重要资源都存放在此。
用于存放系统应用程序，比较重要的目录/usr/local 本地系统管理员软件安装目录(安装系统级的应用)。这是最庞大的目录，要用到的应用程序和文件几乎都在这个目录。

/usr/x11r6 存放x window的目录

/usr/bin 众多的应用程序

/usr/sbin 超级用户的一些管理程序

/usr/doc linux文档

/usr/include linux下开发和编译应用程序所需要的头文件

/usr/lib 常用的动态链接库和软件包的配置文件

/usr/man 帮助文档

/usr/src 源代码，linux内核的源代码就放在/usr/src/linux里

/usr/local/bin 本地增加的命令

/usr/local/lib 本地增加的库

```



# 查看自身的 IP

```Shell
curl cip.cc
```



# ssh免密登录

基本原理：ssh登录分为两种：

- 口令登录

> 客户端发起请求
>
> 服务端发送自己的公钥给客户端
>
> 客户端用 公钥加密密码，发给服务器
>
> 然后服务器用私钥解密，如果密码正确，登录成功

- 密钥登录

> 先把客户端的公钥发送给服务器
>
> 然后客户端发送请求，例如登录请求
>
> 服务器端在授权的密钥列表中查找，找到了，就用客户端的公钥加密一个 自己 随机生成的字符串，发送给客户端
>
> 然后客户端用自己的私钥进行解密，再把解密之后的字符串发给服务器端
>
> 服务器端验证成功后，即登录成功

可以参考这篇，不过注意公钥只能加密，私钥只能解密 [SSH原理与运用（一）：远程登录](https://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

下面是服务器配置免密过程

1、首先客户端生成公钥和私钥，如果没有 .ssh 的话，需要手动生成

```Shell
ssh-keygen -t rsa -C "18365519973@163.com"
```

2、然后把客户端的公钥复制到服务器，相应的用户名的 `.ssh` 下面，例如 `/home/zp_3c/.ssh`。注意不要复制错了，**先打开插入模式**

```Shell
# 如果没有授权文件，需要自己创建
vim ~/.ssh/authorized_keys
```

3、这时候，需要修改 `.ssh` 和 `authorized_keys` 的权限，这俩文件只能让所有者有权限操作，否则验证不通过

```Shell
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

4、如果不能登录的话，看一下配置文件的 `AuthorizedKeysFile` 是否是打开的

```Shell
sudo vim /etc/ssh/sshd_config
# AuthorizedKeysFile 修改成下面的样子
AuthorizedKeysFile      %h/.ssh/authorized_keys
```

5、最后，重启 `ssh` 服务

```Shell
service sshd restart
```



# frp实现通过公网ip访问内网

通过frp搭建隧道，使外网可以直接SSH连接校园网中的服务器，知乎的一篇文章，跟着教程做就行 [参考链接](https://www.zhihu.com/question/22568892)

0.42.0 版本是有 `systemd` 文件的

## 1、准备

1. 固定IP的公网服务器一台（阿里云服务器）
2. 校园网中可SSH连接的实验室服务器若干台（以下步骤需要重复N次）

## 2、步骤

公网服务器作为fcp服务器，校园网服务器作为frp客户端。部署成功后，访问者主机使用SSH命令可直接访问校园网服务器。

- 公网服务器操作

- - 在所需目录下，下载frp压缩包并解压
  - 编辑frp公网服务器配置文件frps.ini
  - 开放公网服务器入站端口（frp服务器服务端口和服务端口）
  - 配置环境，以服务运行
  - 网页登录frp管理服务

- 校园网服务器操作

- - 在所需目录下，下载frp压缩包并解压
  - 编辑frp客户端配置文件frpc.ini
  - 开放公网服务器入站端口（对应每一个frp客户端，frp服务器都需要监听一个对应的端口）
  - 配置环境，以服务运行

### 2.1 公网服务器操作

本人选择阿里云云服务器作为公网服务器，IP为 `120.76.100.14`，请注意替换。

frp工作原理为：

1. 服务端运行，监听一个主端口，等待客户端的连接；
2. 客户端连接到服务端的主端口，同时告诉服务端要监听的端口和转发类型；
3. 服务端fork新的进程监听客户端指定的端口；
4. 外网用户连接到客户端指定的端口，服务端通过和客户端的连接将数据转发到客户端；
5. 客户端进程再将数据转发到本地服务，从而实现内网对外暴露服务的能力。

### 下载frp并解压

在 [frp下载页面](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases) 选择合适版本下载。

在用户目录下创建frp目录，并在其中下载压缩包，并解压进入解压目录。

```Shell
mkdir ~/frp
cd ~/frp
wget https://github.com/fatedier/frp/releases/download/v0.38.0/frp_0.38.0_linux_amd64.tar.gz
tar -zxvf frp_0.38.0_linux_amd64.tar.gz
cd frp_0.38.0_linux_amd64
```

解压目录如下，其中frps系列文件主要用于frp服务器，frpc系列文件主要用于frp客户端（校园网服务器）。

稍后我们在校园服务器上也需要下载同样的软件包。

```Shell
frp_0.38.0_linux_amd64
├── frpc
├── frpc_full.ini
├── frpc.ini
├── frps
├── frps_full.ini
├── frps.ini
├── LICENSE
└── systemd
    ├── frpc.service
    ├── frpc@.service
    ├── frps.service
    └── frps@.service
```

### 编辑frp服务器配置文件frps.ini

```Shell
[common]
# frp监听的端口，默认是7000，可以改成其他的
bind_port = 7000
# 授权码，请改成更复杂的
token = 12345678

# frp管理后台端口，请按自己需求更改
dashboard_port = 7500
# frp管理后台用户名和密码，请改成自己的
dashboard_user = admin
dashboard_pwd = admin
enable_prometheus = true
# 添加日志文件
log_file = /var/log/frps.log
```

### 开放公网服务器入站端口

在云服务器网络设置中，添加以上7000（frp服务器绑定端口）和7500（frp服务管理网页访问端口）TCP协议的入站规则。

### 配置环境，以服务运行

当前路径下，一般可以通过`./frps -c frps.ini`直接开启服务，但是SSH连接断开后就会失效，所以需要借助systemd服务运行。

```Shell
cd ~/frp/frp_0.38.0_linux_amd64/
sudo mkdir -p /etc/frp
sudo cp frps.ini /etc/frp
sudo cp frps /usr/bin
sudo cp systemd/frps.service /usr/lib/systemd/system/
sudo systemctl enable frps
sudo systemctl start frps
```

### 网页登录frp管理服务

顺利的话，可以登录`公网ip:frp服务管理网页访问端口`查看frp的管理界面。下图已经有一个客户端连接。

### 2.2 校园网服务器操作

校园网服务器下载frp并配置客户端文件，以服务运行。

### 下载frp压缩包并解压

同公网服务器。

```shell
mkdir ~/frp
cd ~/frp
wget https://github.com/fatedier/frp/releases/download/v0.38.0/frp_0.38.0_linux_amd64.tar.gz
tar -zxvf frp_0.38.0_linux_amd64.tar.gz
cd frp_0.38.0_linux_amd64
```

### 编辑frp客户端配置文件frpc.ini

```Shell
# 服务端配置
[common]
server_addr = 120.76.100.14
# 请换成设置的服务器端口
server_port = 7000
token = 12345678

# 配置ssh服务,ssh9009是frp连接名，每个frp客户端都要设置不一样的
[ssh9018]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6001  #这个端口就对应了一台实验室服务器的SSH连接，这里把端口和校园网服务器编号保持一致方便记忆
```

### 开放公网服务器入站端口

### 配置环境，以服务运行

```shell
cd ~/frp/frp_0.38.0_linux_amd64/
sudo mkdir -p /etc/frp
sudo cp frpc.ini /etc/frp
sudo cp frpc /usr/bin
sudo mkdir /usr/lib/systemd/system/
sudo cp systemd/frpc.service /usr/lib/systemd/system/
sudo systemctl enable frpc

# Created symlink from /etc/systemd/system/multi-user.target.wants/frpc.service to /usr/lib/systemd/system/frpc.service.

sudo systemctl start frpc
```

### 网页登录frp管理服务验证

## 3、SSH远程登录

登录失败的话，如果报错

```shell
[W] [service.go:104] login to server failed: i/o deadline reached i/o deadli
```

在客户端的（也就是内网服务器）的 `frpc.ini` 的文件中，加上 `tls_enable = true`

如果报错

```shell
login to server failed: connection write timeout
```

[参考这个](https://github.com/fatedier/frp/issues/3193)，应该是异常流量被学校的防火墙给封了，在两边的配置文件里加上

```
tls_enable=true
disable_custom_tls_first_byte = true
```

```Shell
[common]
server_addr = 120.76.100.14
server_port = 7000
token=123456789
tls_enable=true

[ssh6001]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6001
```



## 4. 被学校屏蔽



# Docker部署 frp

服务端和客户端分别拉取两个镜像

```shell
# 服务端 frps
docker pull snowdreamtech/frps

# 客户端 frpc
docker pull snowdreamtech/frpc
```

然后分别找个位置创建 `frp` 文件夹，再创建两个文件

- 服务端： `frps.toml` 和 `restart.sh`
- 客户端： `fprc.toml` 和 `restart.sh`

`frps.toml` 配置

```Shell
[common]
bind_port = 7000
#kcp_bind_port = 7000
# 启用面板
dashboard_port = 7500
# 面板登录名和密码
dashboard_user = vaesong
dashboard_pwd = 1234
# 使用http代理并使用8888端口进行穿透
vhost_http_port = 8888
# 使用https代理并使用9999端口进行穿透
vhost_https_port = 9999
# 日志路径
log_file = ./frps.log
# 日志级别
log_level = info
# 日志最大保存天数
log_max_days = 2
# 认证超时时间
authentication_timeout = 900
# 认证token，客户端需要和此对应
token=123456789
# 最大连接数
max_pool_count = 15
max_ports_per_client = 0

tls_enable=true
disable_custom_tls_first_byte = true
```

服务端 `reatsrt.sh`

```shell
NAME=frps
IMAGE=snowdreamtech/frps

docker stop $NAME
docker rm $NAME

docker run --restart=on-failure:3 --network host -v [frps.toml文件路径]:/etc/frp/frps.ini -d --name $NAME $IMAGE
```

`frpc.toml`

```shell
# 服务端配置
[common]
server_addr = 120.76.100.14
# 请换成设置的服务器端口
server_port = 7000
token = 123456789
tls_enable = true
disable_custom_tls_first_byte = true
#protocol = kcp

# 配置ssh服务,ssh9009是frp连接名，每个frp客户端都要设置不一样的
[ssh6001]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6001
```

客户端 `restart.sh`

```shell
NAME=frpc
IMAGE=snowdreamtech/frpc

sudo docker stop $NAME
sudo docker rm $NAME

sudo docker run --restart=always --network host -v [frpc.toml文件路径]:/etc/frp/frpc.toml -d --name $NAME $IMAGE
```





# ·Linux 删除问题

一定停止所有写入操作！！！还是有机会的，[参考这篇博客](https://blog.csdn.net/qq_40907977/article/details/106761592)



# 防火墙放行端口

[参考这里](https://blog.csdn.net/weixin_44916305/article/details/118355953)

# 深大堡垒机连

```Shell
ssh -o PreferredAuthentications=keyboard-interactive  192750@210.39.9.3
ssh -o PreferredAuthentications=keyboard-interactive  192750#UTF-8#1079#SSH#adminroot@210.39.9.3
# 密码 xiaomi@3c511

# 联网
w3m https://drcom.szu.edu.cn/a70.htm?url=newtab.firefoxchina.cn/newtab/as/activity-stream.html
```



# 命令行向文件写入

通过重定向符`>>`，向文件加入内容，可以使用`echo`，`cat` 等，但是注意，`>`是会进行覆盖的，而 `>>` 不会，谨慎操作。[参考这里](https://www.cnblogs.com/Hackerman/p/15995862.html)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20221101182214.png)

# 挖矿病毒

发现某个进程 `ld-linux-x86-64` 一直在占用 CPU，便把其 kill 了

但是即使 kill 了之后，依然会再次启动，查询得知，是别人的挖矿病毒。[服务器被挖矿入侵](https://blog.csdn.net/xfxfxfxfxf666/article/details/93308360)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//2022-11-07_10-02.png)

1、初步查询，结果如下图

```shell
sudo find / -name ld-linux*
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//2022-11-07_10-09.png)

通过简单分析，`ld-linux-x86-64.so.2` 应该是黑客模拟系统的文件名称，并且还放在 `/tmp/.font-unix` 这个隐藏文件夹下。黑客也有可能写其他跟系统类似的文件，也会放在其他用户比较熟悉的文件夹名称下，混淆视听。

注意这里的通常是 `/tmp`，`dev` 等文件夹下面，`usr` 下面的通常都是一些系统的链接库等，一般不能删除，当然不排除被黑客入侵到了那里

2、查看是否有定时任务

大部分挖矿木马会通过在受感染主机中写入计划任务实现持久化，如果仅仅只是清除挖矿进程，无法将其根除，到了预设的时间点，系统会通过计划任务从黑客的C2服务器重新下载并执行挖矿木马。

查看指定用户的定时任务，发现果然有可疑的定时任务

```Shell
sudo crontab -u tianyu -l
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//2022-11-07_10-36.png)

到这里就可以确定是被植入了挖矿木马，删除 `/tmp/.font-unix/CVE-2021-4034-main` 下的 `lan` 文件夹就行了

在 `/var/spool/cron` 下面，查看是否有恶意仿造的文件，是否存在非法定时任务脚本，并清理

![](https://cdn.jsdelivr.net/gh/vaesong/Images//2022-11-07_13-19.png)

- 1>/dev/null 首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。
-  2>&1 接着，标准错误输出重定向等同于标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。
- 总的来说，就是不会在终端显示任何输出信息以及错误信息

3、继续排查后续

[云服务器被植入挖矿木马，CPU飙升200%](https://blog.csdn.net/qq_23930765/article/details/121105205)

排查一下被入侵用户的 `.ssh` 下面是否被授权免密登录了

```Shell
# 被入侵用户的.ssh
~/.ssh/authorized_keys
```

还要排查以下目录及文件，及时删除可疑的启动项。排查的时候，可以按照文件修改时间来排序，重点排查近期被创建服务项

```Shell
ll -t /usr/lib/systemd/system
ll -t /usr/lib/systemd/system/multi-user.target.wants
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//2022-11-07_13-07.png)

4、后续，查看用户登录

```Shell
last
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//2022-11-07_13-42.png)

# 防火墙问题

## firewall

1、查看firewall服务状态

```
systemctl status firewalld
```

出现`Active: active (running)`切高亮显示则表示是启动状态。

出现`Active: inactive (dead)`灰色表示停止，看单词也行。

2、查看firewall的状态

```
firewall-cmd --state
```

3、开启、重启、关闭、firewalld.service服务

\# 开启

```shell
service firewalld start
```

\# 重启

```
service firewalld restart
```

\# 关闭

```
service firewalld stop
```

4、查看防火墙规则

```
firewall-cmd --list-all
```

5、查询、开放、关闭端口

\# 查询端口是否开放

```
firewall-cmd --query-port=8080/tcp
```

\# 开放80端口

```
firewall-cmd --permanent --add-port=80/tcp
```

\# 移除端口

```
firewall-cmd --permanent --remove-port=8080/tcp
```

\#重启防火墙(修改配置后要重启防火墙)

```
firewall-cmd --reload
```

参数解释

1、`firwall-cmd`：是Linux提供的操作firewall的一个工具；

2、`permanent`：表示设置为持久；

3、`add-port`：标识添加的端口；

## ufw

1、查看防火墙当前状态

```shell
systemctl status ufw
```

2、开启防火墙

```Shell
sudo ufw enable
```

3、关闭防火墙

```Shell
sudo ufw disable
```

4、允许某个IP地址访问本机所有端口

```Shell
sudo ufw allow from 120.76.100.14
```

5、允许外部访问某个端口

```Shell
sudo ufw allow 80
```

6、要删除一条规则

```Shell
sudo ufw delete allow 80
```



# history命令

当用户登录时候，这次登录执行的命令会存放到系统的缓冲区中

使用 `history` 命令可以查看本次登录的所有历史命令

当用户退出登录的时候，系统会自动把缓冲区的命令存放到指定的文件中去，通常是 `~/.bash_history` 

想要清空当次系统缓冲区的命令，这样就不会留下记录，`history -c` 

设置时间格式，但是保存到文件中格式好像没变

```Shell
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
DT=`date +"%Y%m%d_%H:%M:%S"`
export HISTTIMEFORMAT="${USER}@${USER_IP}_$DT "
```



# w3m访问不能解码

访问的时候出现 `gzip: stdin: not in gzip format` 导致无法正常的访问网站

在查询的时候传递接受的编码方式

```Shell
w3m www.baidu.com -o accept_encoding='identity;q=0'
```



# 包的依赖问题

The following packages have unmet dependencies

使用 `aptitude` 来安装

```Shell
sudo apt-get install aptitude
sudo aptitude install XXXXX 
```



# 查看当前启动方式

如果启动方式为 graphical.target ，则表示默认启动方式为进入图形界面

```Shell
systemctl get-default
```



# Isaac sim(docker + client)

```python
kit = SimulationApp(launch_config=CONFIG)
ext_manager = omni.kit.app.get_app().get_extension_manager()
kit.set_setting("/app/window/drawMouse", True)
kit.set_setting("/app/livestream/proto", "ws")
ext_manager.set_extension_enabled_immediate("omni.kit.livestream.core", True)
ext_manager.set_extension_enabled_immediate("omni.kit.livestream.native", True)
```

```Python
from omni.isaac.kit import SimulationApp

# This sample enables a livestream server to connect to when running headless
CONFIG = {
    "width": 1280,
    "height": 720,
    "window_width": 1920,
    "window_height": 1080,
    "headless": True,
    "renderer": "RayTracedLighting",
    "display_options": 3286,  # Set display options to show default grid
}


# Start the omniverse application
kit = SimulationApp(launch_config=CONFIG)

from omni.isaac.core.utils.extensions import enable_extension

# Default Livestream settings
kit.set_setting("/app/window/drawMouse", True)
kit.set_setting("/app/livestream/proto", "ws")
kit.set_setting("/app/livestream/websocket/framerate_limit", 120)
kit.set_setting("/ngx/enabled", False)

# Note: Only one livestream extension can be enabled at a time
# Enable Native Livestream extension
# Default App: Streaming Client from the Omniverse Launcher
enable_extension("omni.kit.livestream.native")

# Enable WebSocket Livestream extension
# Default URL: http://localhost:8211/streaming/client/
# enable_extension("omni.services.streamclient.websocket")

# Enable WebRTC Livestream extension
# Default URL: http://localhost:8211/streaming/webrtc-client/
# enable_extension("omni.services.streamclient.webrtc")

# Run until closed
while kit._app.is_running() and not kit.is_exiting():
    # Run in realtime mode, we don't specify the step size
    kit.update()

kit.close()
```



# VNC 配置远程连接

首先[参考链接](https://www.mintimate.cn/2021/05/15/installVNC/)

xfce什么的就不用装了，已经装过 gnome 就行

主要的是 `vncserver` 的配置，在 `~/.vnc/xstartup` 下面，之前的都是黑屏什么的。最主要的是 [参考这篇](https://blog.csdn.net/f066314/article/details/126745810)

```Shell
#!/bin/sh
export XKL_XMODMAP_DISABLE=1
export XDG_CURRENT_DESKTOP="GNOME-Flashback:GNOME"
export XDG_MENU_PREFIX="gnome-flashback-"
gnome-session --session=gnome-flashback-metacity --disable-acceleration-check &
```



# Todesk 虚拟屏

安装虚拟屏软件

```Shell
sudo apt-get install  xserver-xorg-core-hwe-18.04
sudo apt-get install  xserver-xorg-video-dummy
```



。。。。太麻烦了，其实主要是使用了自动登陆或者其他原因吧，系统使用Wayland窗口时todesk等会获取不到屏幕，在开机登陆界面右下角的小齿轮那里把Wayland切换回xorg就可以了，也可以手动禁用Wayland，/etc/gdm3/custom.conf取消注释即可
\#WaylandEnable=false。
注销或者重启后进入
设置-关于-窗口系统，显示为x11即可，再次用todesk连接，已经有桌面了



# Ubuntu 命令打开系统设置

```Shell
gnome-control-center
```





# libGL error: failed to load driver: swrast

xvfb 报错出现问题，libGL error: MESA-LOADER: failed to open swrast: /lib/x86_64-linux-gnu/libLLVM-12.so.1: undefined symbol: ffi_type_sint32, version LIBFFI_BASE_7.0 (search paths /usr/lib/x86_64-linux-gnu/dri:\$${ORIGIN}/dri:/usr/lib/dri, suffix _dri)

通过 `new bing` 搜索，得到答案，[参考博客](https://stackoverflow.com/questions/40092796/libgl-error-failed-to-load-driver-swrast-running-ubuntu-docker-container-on)

**注意：但是不行！！！！**

```Shell
sudo apt-get install -y mesa-utils libgl1-mesa-glx
```

仔细分析报错发现，是需要 `/lib/x86_64-linux-gnu` 下的 `libLLVM-12.so.1` 这个动态库，但是这个动态库依赖 `LIBFFI_BASE_7.0`，也就是 `libffi.so.7`，但是在搜索的路径 `/usr/lib/x86_64-linux-gnu/dri:\$${ORIGIN}/dri:/usr/lib/dri, suffix _dri` 中并没有找到。

使用命令查看动态库的依赖，发现所有依赖的动态库都是可以找得到的

```shell
ldd -r libLLVM-12.so.1
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230314170459.png)

而且 `libffi.so.7` 在该文件夹下是可以找到的，于是想着把它移动到 `/usr/lib/x86_64-linux-gnu/dri:\$${ORIGIN}/dri:/usr/lib/dri, suffix _dri` 下面，但是并不行

后面查到可能是动态库路径设置的问题，[Linux：静态库和动态库](https://blog.csdn.net/qq_45481606/article/details/119335681), [ImportError: /lib/aarch64-linux-gnu/libwayland-server.so.0: undefined symbol: ffi_type_uint32...](https://blog.csdn.net/m0_57315535/article/details/128113311) 

然后首先用的 `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib/x86_64-linux-gnu` 直接配置动态库的路径，结果还是不行

最后在第二篇文章下解决 `export LD_PRELOAD=/lib/x86_64-linux-gnu/libffi.so.7` 。



# find 查找指定的路径，但是排除某些路径

例如想查找根目录，但是不想查 `/data1` 和 `data2` 

```Shell
# 添加选项 -path /data1 -prune -o
sudo find / -path /data1 -prune -o -path /data2 -prune -o -user lsy
```





# linux 如何查找进程的执行路径

首先查出进程号

```Shell
ps -aux|grep "command"
```



得到进程号之后通过查看/proc 获取进程执行路径

```Shell
ll /proc/pid/cwd
```





# SSH 暴力破解 IP

查看密码是否被修改

```Shell
ll /etc | grep shadow
```



查看进程，因为是暴力破解，那么一般都需要 `ssh` 命令

```Shell
ps -aux | grep sshd
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230327174338.png)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230410185016.png)

然后发现确实存在大量的 `sshd` 进程，命令是 `sshd iplist passbombalol 22 nproc`，后续查找才发现这条命令的意思，就是使用 `sshd` ，`iplist` 是扫描得到的 ip 地址保存的文件。`passbombalol` 文件存放的是大量的用户名和密码字典

```Shell
sudo find / -name iplist
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230327222516.png)

```Shell
cd /var/tmp/a
ll
```

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230327222622.png)

查看这两个文件

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230327222730.png)

![](https://cdn.jsdelivr.net/gh/vaesong/Images//20230327222758.png)

```Shell
killall -u xhx
```



# Linux 更新 CUDA , 不用安装驱动

[非root用户在linux下安装多个版本的CUDA和cuDNN（cuda 8、cuda 10.1 等）](https://blog.csdn.net/hizengbiao/article/details/88625044)

[ubantu下非root用户安装CUDA和cuDNN——以CUDA11.6为例](https://zhuanlan.zhihu.com/p/614819188)

方法流程

1. 下载和 `nvidia` 驱动匹配的 `CUDA Toolkit` 版本
2. 下载和 `CUDA Toolkit` 版本匹配的 `cudnn` 
3. 安装 `CUDA Toolkit` 
4. 把 `cudnn` 的 `include` 和 `lib` 文件夹下的文件复制到 `CUDA Toolkit` 对应的文件夹下，并修改权限
5. 配置路径



# 进入docker 使用root用户的方式

```shell
docker exec -it --user root <container id> bash
```



# 重启网络

重启网络服务是另一种重启网络的方式

```Shell
sudo service networking restart
```

如果使用的是systemd服务管理器，可以使用以下命令重启网络服务：

```shell
sudo systemctl restart networking.service
```

如果使用的是network-manager在Debian上

```shell
sudo systemctl restart NetworkManager.service
```

